---
title: redis主从+keepalived
date: 2018-12-12 07:52:04
password: passw0rd
tags:
---

为保证redis在运行中应对宕机故障，采用Redis主从+KeepAlived实现高可用。此次试验事先关闭了防火墙和selinux，生产环境建议开启防火墙。

172.20.100.189 主redis  主keepalived
172.20.100.191 从redis  从keepalived

VIP   172.20.100.66 （随意，一般同网段只要没人在用即可） --------------应用（程序配置）通过VIP的6379端口访问redis数据库

redis-4.0.8
Keepalived-2.0.8                    keepalived直接用yum装也可以

原理（思路）： 
正常运行下，主负责服务，从负责同步，应用通过vip的6379端口访问redis数据库，当主节点189宕机（或服务挂掉）后， vip漂到191上，191会接管189成为主节点，继续提供读写操作，同时关闭主从复制功能。
当主恢复后，主会和从进行一次数据同步（即从slave同步数据），同步完成后恢复master身份，而Slave等Master同步完数据后会再恢复到Slave身份，继续不间断的读写服务。


有时候我们需要M服务器在恢复正常后不要重新接管VIP，让B服务器继续为主  让后来恢复正常的M服务器为备。如下处理：   

修改Master配置， Backup服务器的配置不变。
state MASTER 修改为 state BACKUP 
nopreempt  添加该参数，设置为不抢夺VIP

主从都为slave类型，同时主设置不抢夺vip，然后先后启动主从，因为主的优先级高，所以从不会抢夺vip。当主挂掉后，从成为主，然后等原来主恢复后，由于设置了不抢夺vip，所以从继续为主。


-----------------------------具体操作如下-----------------------------------


检查Java，没有的话进行安装
java -version
配置java环境变量vi /etc/profile
```
export JAVA_HOME=/usr/local/java/jdk1.8.0_171
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```
#安装依赖
```
yum -y install gcc-c++
```
#下载redis包，先浏览器访问http://download.redis.io/releases/ 看看用哪个，然后下载即可
```
cd /usr/local/src
wget http://download.redis.io/releases/redis-4.0.8.tar.gz
tar zxvf redis-4.0.8.tar.gz -C /usr/local/
mv redis-4.0.8 redis
cd /usr/local/redis
#编译安装
makeMALLOC=libc
make PREFIX=/usr/local/redis install
```
#配置redis启动脚本
vi /etc/init.d/redis
```
#!/bin/sh
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
#配置redis端口号
REDISPORT=6379
#配置redis启动命令路径
EXE=/usr/local/redis/bin/redis-server
#配置redis连接命令路径
CLIEXE=/usr/local/redis/bin/redis-cli
#配置redis运行PID路径
PIDFILE=/var/run/redis_6379.pid
#配置redis的配置文件路径
CONF="/etc/redis/redis.conf"
#配置redis的连接认证密码
REDISPASSWORD=123456

function start () {
        if [ -f $PIDFILE ]

        then

                echo "$PIDFILE exists,process is already running or crashed"

        else

                echo "Starting Redisserver..."

                $EXE $CONF &

        fi
}

function stop () {
        if [ ! -f $PIDFILE ]

        then

                echo "$PIDFILE does not exist, process is not running"

        else

                PID=$(cat $PIDFILE)

                echo "Stopping ..."

                $CLIEXE -p $REDISPORT -a $REDISPASSWORD shutdown

                while [ -x /proc/${PID} ]

                do

                    echo "Waiting forRedis to shutdown ..."

                    sleep 1

                done

                echo "Redis stopped"

        fi
}

function restart () {
        stop
        
        sleep 3
        
        start
}

case "$1" in
    start)
    start
    ;;
    stop)
    stop
    ;;
    restart)
    restart
    ;;
    *)
    echo -e "\e[31m Please use $0 [start|stop|restart] asfirst argument \e[0m"
    ;;
esac
```
赋予权限并添加开机自启动
```
chmod +x /etc/init.d/redis
chkconfig --add redis
chkconfig redis on
```
#添加redis命令环境变量
```
vi /etc/profile
export PATH="$PATH:/usr/local/redis/bin"   #添加到末尾即可
source /etc/profile  #配置生效
```

#配置redis的配置文件vim /etc/redis/redis.conf
master配置如下：
```
bind 0.0.0.0
port 6379
daemonize yes
requirepass 123456            #设置认证密码，如果无需要则取消
slave-serve-stale-data yes
slave-read-only no
pidfile /var/run/redis_6379.pid
logfile /var/log/redis.log
```

slave配置如下:
```
bind 0.0.0.0
port 6379
daemonize yes
slaveof 172.20.100.189 6379    #注意这里是主节点的ip
masterauth 123456
slave-serve-stale-data yes
slave-read-only no
pidfile /var/run/redis_6379.pid
logfile /var/log/redis.log
```
配置完成后重启redis服务!!!

验证主从是否正常：
主节点189终端登录测试：
[root@localhost ~]# redis-cli -a 123456
127.0.0.1:6379> info


主节点191终端登录测试：
[root@localhost ~]# redis-cli -a 123456
127.0.0.1:6379> info

然后再看看测试截图


-------------------------------------到此redis的主从已经完成！


接下来！！！ 
开始配置KeepAlived实现双机热备

#安装，有两种方式-----这里用的是yum安装
1.yum安装yum -y install keepalived


默认安装有可能是没有配置文件的。新建一个 vi /etc/keepalived/keepalived.conf 注意主备

---master配置如下：
```
vrrp_script chk_redis {
    script "/etc/keepalived/script/redis_check.sh"     # 检查redis的脚本
    interval 2                          # 每两秒检查一次
    weight 2                            #每次检查的加权权重值
}


vrrp_instance VI_1 {
    state MASTER                #指定实例初始化的状态,如果都是backup，那么就按照priority的值来确定谁是master
    interface eth0              #网卡
    virtual_router_id 61        # VRID标记（0-255）
    priority 101                #priority 高优先级的为master,最好相差大于50
    nopreempt                   #设置不抢占  --如果这样的话，上面的state MASTER 也要修改为 state BACKUP 
    advert_int 1                # 检查间隔时间,默认1s

    authentication {                 #authentication 这一段设置认证
        auth_type PASS               #认证方式，支持PASS和HA（据说HA有问题）
        auth_pass 1111               #认证密码
    }


    virtual_ipaddress {
        172.20.100.66                    #VIP
    }


    track_script {
        chk_redis                     # 调用检查脚本
    }

    notify_master /etc/keepalived/script/redis_master.sh
    notify_backup /etc/keepalived/script/redis_backup.sh
    notify_fault  /etc/keepalived/script/redis_fault.sh  
    notify_stop  /etc/keepalived/script/redis_stop.sh

}

```

----slave配置如下:
```

vrrp_script chk_redis {
    script "/etc/keepalived/script/redis_check.sh"     # 检查haproxy的脚本
    interval 2                          # 每两秒检查一次
}


vrrp_instance VI_1 {
    state BACKUP                #指定实例初始化的状态,如果都是backup，那么就按照priority的值来确定谁是master
    interface eth0              #网卡
    virtual_router_id 61        # VRID标记（0-255）
    priority 99                #priority 高优先级的为master,最好相差大于50
    advert_int 1                # 检查间隔时间,默认1s

    authentication {                 #authentication 这一段设置认证
        auth_type PASS               #认证方式，支持PASS和HA（据说HA有问题）
        auth_pass 1111               #认证密码
    }

    virtual_ipaddress {
        172.20.100.66                  #VIP
    }

    track_script {
        chk_redis                     # 调用检查脚本
    }

    notify_master /etc/keepalived/script/redis_master.sh
    notify_backup /etc/keepalived/script/redis_backup.sh
    notify_fault  /etc/keepalived/script/redis_fault.sh  
    notify_stop  /etc/keepalived/script/redis_stop.sh

}
```

########配置脚本--此为负责运作的关键脚本
Master KeepAlived---189
#创建存放脚本目录：
mkdir -p /etc/keepalived/script
cd /etc/keepalived/script
编辑脚本，共5个
```
1.vi redis_backup.sh

#!/bin/bash 

REDISCLI="/usr/local/redis/bin/redis-cli -a 123456"

LOGFILE="/var/log/keepalived-redis-state.log"

echo "[backup]" >> $LOGFILE

date >> $LOGFILE

echo "Being slave...." >>$LOGFILE 2>&1

sleep 15 #延迟15秒待数据被对方同步完成之后再切换主从角色 

echo "Run SLAVEOF cmd ...">> $LOGFILE

$REDISCLI SLAVEOF 172.20.100.191 6379 >>$LOGFILE  2>&1

2. vi redis_check.sh
#!/bin/bash 

ALIVE=`/usr/local/redis/bin/redis-cli -a 123456 PING` 

if [ "$ALIVE" == "PONG" ];then

echo $ALIVE 

exit 0 

else

echo $ALIVE 

exit 1 

fi

3. vi redis_fault.sh
#!/bin/bash 

ALIVE=`/usr/local/redis/bin/redis-cli -a 123456 PING` 

if [ "$ALIVE" == "PONG" ];then

echo $ALIVE 

exit 0 

else

echo $ALIVE 

exit 1 

fi
[root@myself script]# more redis_fault.sh
#!/bin/bash

LOGFILE=/var/log/keepalived-redis-state.log

echo "[fault]" >> $LOGFILE

date >> $LOGFILE

4. vi redis_master.sh
#!/bin/bash
REDISCLI="/usr/local/redis/bin/redis-cli -a 123456"
LOGFILE="/var/log/keepalived-redis-state.log"

sleep 15

echo "[master]" >> $LOGFILE

date >> $LOGFILE

echo "Being master...." >>$LOGFILE 2>&1


echo "Run SLAVEOF cmd ...">> $LOGFILE

$REDISCLI SLAVEOF 172.20.100.191 6379 >>$LOGFILE  2>&1
if [ $? -ne 0 ];then
    echo "data rsync fail." >>$LOGFILE 2>&1
else
    echo "data rsync OK." >> $LOGFILE  2>&1
fi

sleep 10 #延迟10秒以后待数据同步完成后再取消同步状态 

echo "Run SLAVEOF NO ONE cmd ...">> $LOGFILE

$REDISCLI SLAVEOF NO ONE >> $LOGFILE 2>&1
if [ $? -ne 0 ];then
    echo "Run SLAVEOF NO ONE cmd fail." >>$LOGFILE 2>&1
else
    echo "Run SLAVEOF NO ONE cmd OK." >> $LOGFILE  2>&1
fi

5. vi redis_stop.sh
#!/bin/bash

LOGFILE=/var/log/keepalived-redis-state.log

echo "[stop]" >> $LOGFILE

date >> $LOGFILE

```

Slave KeepAlived---191
#创建存放脚本目录：
mkdir -p /etc/keepalived/script
cd /etc/keepalived/script

----script里面只需要更改redis_master.sh和redis_backup.sh里面的ip即可，fault和stop的脚本 主从一样


///注解：

Keepalived在转换状态时会依照状态来呼叫:

当keepalived进入Master状态时，会执行notify_master; 

当keepalived进入Backup状态时，会执行notify_backup;

当keepalived发现异常情况时进入进入fault状态时，会执行notify_fault；

当Keepalived程序终止进入stop状态时，会执行notify_stop；


///









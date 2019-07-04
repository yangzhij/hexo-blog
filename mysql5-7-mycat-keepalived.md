---
title: mysql5.7+mycat+keepalived
date: 2019-05-20 09:46:36
password: passw0rd
tags:
---
mysql主从
```
主库负责增删改操作。从库负责查询。
	原理：
		master将操作语句（数据的改变）记录到binlog日志中（在每个事务更新数据完成之前），然后授予slave远程连接的权限。（前提：master一定要开启binlog二进制日志功能）
		slave开启两个线程：IO线程和SQL线程。IO线程负责读取master的binlog内容到中继日志relay log里；SQL线程负责从relay log日志里读出binlog内容，并更新到slave的数据库里。（这样来保证主从数据的一致）
	要求：
		主从版本一致
		节点时间一致
	优点：
		降低主服务器压力;（主库写，从库读，降压）
		在从库进行备份，避免备份期间影响主库服务;（确保数据安全）
		主从切换（提升性能）
	主从部署必要条件：
		主库开启binlog日志（设置log-bin参数）
		从库服务器能连通主库
		主从server-id不同
	
	半同步复制---解决数据丢失的问题，通过安装插件
	并行复制----解决从库复制延迟的问题，通过在my.cnf里配置参数
		
主从与主主模式:
	主从复制：主库授权从库远程连接，通过读取主库binlog并更新到本地数据库的过程。从库跟着主库Centos7下mysql5.7+mycat+keepalived变。
	主主复制：主从相互授权连接，读取对方binlog日志并更新到本地数据库的过程。双方都随着对方的改变而改变。

当从库的IO和SQL线程的状态均为Yes，则表示主从已实现同步了！
```

//部署如下
```
cd /usr/local/src/
#官网下载mysql-server
wget http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
#mysql依赖包
yum -y install mysql57-community-release-el7-10.noarch.rpm
#安装mysql数据库
yum -y install mysql-community-server
#安装sysbench基准测试工具，依赖于上述mysql依赖包
yum install sysbench
#启动
systemctl start mysqld
# 设置开机启动
systemctl enable mysqld
systemctl daemon-reload
#在日志文件中查看初始密码
grep "password" /var/log/mysqld.log
#进入修改密码，必须先修改，否则啥也不能做，可能会提示密码过于简单之类的报错，先设置个复杂的，然后再修改密码规则
mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'P@ssw0rd123';

#查看mysql密码规则
mysql> SHOW VARIABLES LIKE 'validate_password%';
#修改MySQL默认策略和密码长度（mysql5.7默认安装了密码安全检查插件（validate_password），默认密码检查策略要求密码必须包含：大小写字母、数字和特殊符号，并且长度不能少于8位。）
mysql> set global validate_password_policy=0;
mysql> set global validate_password_policy=LOW;
mysql> SET GLOBAL validate_password_length=6;

#之后就可以改成简单密码了
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

#卸载安装源自动更新
yum -y remove mysql57-community-release.noarch

#初始化数据库
[root@localhost ~]# mysql_secure_installation
Enter password for user root:    #需输入mysql密码
一路回车
Disallow root login remotely?    看情况是否允许root用户远程访问


6.进入数据库设置远程登录用户，不一定要是root
mysql> grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
mysql> flush privileges;



配置主mysql数据库配置文件：vim /etc/my.cnf
    1、配置文件配置完之后，登录mysql数据库设置数据同步权限
        mysql> grant replication slave,replication client on *.* to root@'172.20.100.122' identified by "wwwmysqlpw;
        mysql>flush privileges;
		
    2、查看master服务器状态
        mysql> show master status;(注意File与Position项，从服务器需要这两项参数)	

配置从mysql配置文件:vim /etc/my.cnf，只需要变更一下配置中的server-id,
#添加一下
slave-skip-errors = all #跳过所有的错误错误，继续执行复制操作
	配置主从同步指令
mysql>stop slave;   ＃执行同步前，要先关闭slave
mysql>change  master to master_host='172.20.100.121',master_user='root',master_password='wwwmysqlpw',master_log_file='mysql-bin.000006',master_log_pos=110;	
mysql>start slave;          #开启同步
mysql>show slave status \G

当IO和SQL线程的状态均为Yes，则表示主从已实现同步了！

主从同步是基于主从配置完之后开始，并且mysql主从部署之后可能会出现主从同步延迟的问题
```

---mycat安装
```
cd /usr/local/src/
wget http://dl.mycat.io/1.6-RELEASE/Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz
tar -zxvf Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz
chmod -R 777 mycat
添加环境变量
vi /etc/profile
export MYCAT_HOME=/usr/local/src/mycat
export PATH=$PATH:$MYCAT_HOME/bin
使环境变量生效
source /etc/profile


mycat配置文件
cd /usr/local/src/mycat/conf
schema.xml       数据库设置，分库分表规则
server.xml	    用户权限设置，与mysql无关


启动mycat
cd /usr/local/src/mycat/bin/
sh mycat start
```

用户授权
```
mysql> grant select,update,delete,insert on micro_game.* to test@'%' identified by 'test@123456';
mysql> flush privileges; 
```

mysql半同步复制配置
```
主库配置
mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
mysql> set global rpl_semi_sync_master_enabled=on;

从库配置
mysql>  INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
mysql>  set global rpl_semi_sync_slave_enabled=on;

加入my.cnf参数
主库[mysqld]添加：
rpl_semi_sync_master_enabled = 1
从库[mysqld]添加：
rpl_semi_sync_slave_enabled = 1


在主从上验证一下
mysql> show global status like 'rpl_semi%';
Rpl_semi_sync_master_status                | ON    |
Rpl_semi_sync_slave_status | ON    |
```

程序配置文件：
```
database=micro_game
server=172.20.100.247:8066   --注意提供的是mycat的端口,因为只有通过mycat才能实现读写分离，帮你转发
username=test
password=test@123456
```
---
title: rsync+inotify 实现文件实时同步
date: 2019-06-22 14:58:45
tags:
---

**rsync**
rsync首次同步数据时会把全部文件拷贝一次，使本地和远程两个主机之间的文件达到同步。之后再同步时，会扫描所有文件后进行比对，然后进行差量传输。
rsync具有安全性高、备份迅速、支持增量备份。可以解决对实时性要求不高的数据备份需求。如定期地备份文件服务器数据到远程服务器上，对本地磁盘定期进行数据镜像等。但，rsync不能够实时监测、同步数据。如果有这方面要求，这就需要和inotify组合，来实现数据的实时同步。

//提炼
```
同步本地文件：rsync -avz /data /backup
同步本地文件到远程：
rsync -avz /data test@192.168.199.247:/backup
rsync -avz /data test@192.168.199.247::backup –password-file=/etc/rsyncd.password（常用推送方式）

把远程机器的文件同步到本地：
rsync -avz test@192.168.199.247:/backup /data
rsync -avz test@192.168.199.247::backup –password-file=/etc/rsyncd.password /data  （常用拉取方式）

服务端配置：
rsync服务器端需要两个配置文件：rsyncd.conf（主配置）、rsyncd.password（密码文件，存储rsync用户名和密码）；
但是在rsync安装完毕后后是不会生成以上这两个配置文件的，需要我们手工进行创建。

通过rsync可以解决对实时性不高的数据备份需求。
缺点：首先rsync同步数据时需要扫描所有文件后进行对比，然后差量传输，对于大数据这是非常低效的。
其次，rsync不能时时的去检测，同步数据。基于以上原因，考虑采用rsync+inotify解决这些问题。
```
	
**项目需求**
把公司热更服务器上的/home/wwwroot下所有目录及文件备份至文件服务器/home/wwwroot，要求实现目录实时同步。

热更服务器 172.20.100.155
文件服务器 172.20.100.115
	
**文件服务器操作**
//安装：
```
检查rsync 是否已经安装：rpm -qa|grep rsync；若已经安装，则使用rpm -e 命令卸载(如果提示有依赖，可以加上 --nodeps)。
yum -y install gcc
wget https://download.samba.org/pub/rsync/rsync-3.1.3.tar.gz 
tar -xzf rsync-3.1.3.tar.gz
cd rsync-3.1.3
./configure
make && make install
/usr/local/bin/rsync --daemon     //启动
设置开机启动：echo “/usr/local/bin/rsync –daemon”>>/etc/rc.local
```

//编辑配置文件
```
vi /etc/rsyncd.conf
uid = root
gid = root
use chroot = yes
max connections = 10
strict mode=yes
pid file = /var/run/rsyncd.pid
lock file=/var/run/rsync.lock
log file=/var/log/rsyncd.log
[web1]   //模块名
path = /home/wwwroot               //这个目录手动创建   同步的内容会放到这个目录下
comment = web1 file
ignore errrors
read only=no
write only=no
hosts allow=*           //允许所有主机
hosts deny=192.168.100.10    //拒绝某台主机
list=false
uid=root
gid=root
auth users=web1user          //允许用户
secrets file=/etc/web1.pass
```
//创建密码文件
```
vi /etc/web1.pass    格式如下

web1user:admin123
```
//修改密码文件权限
chmod 600 /etc/web1.pass


**热更服务器操作**
```
//安装
yum -y install gcc
cd /usr/local/src
wget https://download.samba.org/pub/rsync/rsync-3.1.3.tar.gz 
tar zxvf rsync-3.1.3.tar.gz -C /opt
cd /opt/rsync-3.1.3
./configure
make && make install

生成rsync的密码文件
vim /etc/server.pass   直接把密码放里面，保存即可

admin123

//权限
chmod 600 /etc/server.pass
```

**先测试一下~~~**
把热更上/home/wwwroot   同步到远程机器上
·```
rsync -vzrtopg --progress --delete /home/wwwroot web1user@172.20.100.115::web1 --password-file=/etc/server.pass

//解释
--progress是指显示出详细的进度情况
--delete是指如果服务器端删除了这一文件，那么客户端也相应把文件删除，保持真正的一致。
--password-file来指定密码文件
web1user@172.20.100.115::web1    用户名@ip::模块名
```

//inotify 下载安装
```
wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz
tar zxvf inotify-tools-3.14.tar.gz -C /opt
cd -C /opt/inotify-tools-3.14
cd /opt/inotify-tools-3.14
./configure
make && make install
```
//修改内核的inotify参数
```
vi /etc/sysctl.conf 

fs.inotify.max_user_instances = 65535
fs.inotify.max_user_watches = 6553500         监控的文件数量很大时，对应的数值需要修改
fs.inotify.max_queued_events = 6553500

sysctl -p       #让上述修改立刻生效 
```

//写脚本来监控热更机器/home/wwwroot/ 目录
```
vim /web/inotifyrsync.sh

#!/bin/bash
host1=172.20.100.155         // 热更服务器
src=/home/wwwroot/           // 监控目录
mokuai=web1
user1=web1user
/usr/local/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e close_write,delete,create,attrib $src \
| while read files
do
        rsync -vzrtopg --progress --delete $src --password-file=/etc/server.pass  $user1@$host1::$mokuai > /dev/null 2>&1
        echo "${files} was rsynced." >> /tmp/rsync.log 2>&1
done
```

//测试脚本
```
chmod 755 /web/inotifyrsync.sh
/web/inotifyrsync.sh &
```
//开机启动脚本
```
echo "/web/inotifyrsync.sh &" >> /etc/rc.local
```
然后测试下看能否同步......












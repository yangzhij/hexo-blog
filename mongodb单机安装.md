---
title: mongodb 单机安装
date: 2018-12-22 18:32:21
tags:
---

**MongoDB是一个基于分布式文件存储的数据库，其目的在于为WEB应用提供可扩展的高性能数据存储解决方案。**
此为单机部署mongodb, 直接操作此文档命令即可………………………
软件下载https://fastdl.mongodb.org，可以看看目前版本，然后直接在下面改下版本号即可
安装过程如下：
```
cd /usr/local
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.0.4.tgz
tar xvf mongodb-linux-x86_64-rhel70-4.0.4.tgz
mkdir mongodb
mv mongodb-linux-x86_64-rhel70-4.0.4 mongodb/mongodb4.0.4
cd mongodb/mongodb4.0.4
mkdir conf logs
mkdir -p data/db
```
**conf 存放配置文件的目录  logs 存放日志的目录   data/db存放数据文件的目录 **
```
cd conf/
vi mongodb.conf （复制下列内容即可）
#idae - MongoDB config start - 2018-11-27
#设置数据文件的存放目录
dbpath = /usr/local/mongodb/mongodb4.0.4/data/db
#设置日志文件的存放目录及其日志文件名
logpath = /usr/local/mongodb/mongodb4.0.4/logs/mongodb.log
# 设置端口号
port = 27017
#设置运行任意ip连接
bind_ip = 0.0.0.0
# 设置为以守护进程的方式运行，即在后台运行
fork = true
保存退出
```
#启动
```
cd ../bin
./mongod --config ../conf/mongodb.conf
```
启动完毕后，用客户端连接，没有用户名密码


**加入开机自启动..将 mongodb 服务加入到自启动文件中,即在文件末尾追加如下命令:**
```
vi /etc/rc.local 
/usr/local/mongodb/mongodb4.0.4/bin/mongod --config /usr/local/mongodb/mongodb4.0.4/conf/mongodb.conf
chmod +x /etc/rc.local   #执行权限
```


如下一行命令搞定
```
echo /usr/local/mongodb/mongodb4.0.4/bin/mongod --config /usr/local/mongodb/mongodb4.0.4/conf/mongodb.conf >>/etc/rc.local
chmod +x /etc/rc.local
```
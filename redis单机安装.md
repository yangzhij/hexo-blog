---
title: redis单机安装
date: 2018-12-16 09:05:47
tags:
---
先对系统优化一下吧，先参考linux基本优化配置文档。
1.浏览器查看下载哪个版本http://download.redis.io/release
2.安装依赖 yum -y install gcc
3.安装
```
cd /usr/local
wget http://download.redis.io/releases/redis-5.0.1.tar.gz
tar zxvf redis-5.0.1.tar.gz 
cd redis-5.0.1
make MALLOC=libc
cd src && make install
```
4.更改基本配置： #以守护进程方式启动     #允许所有ip远程访问（中间两个）   #有必要的情况下修改默认端口  
```
sed -i 's/daemonize no/daemonize yes/' /usr/local/redis-5.0.1/redis.conf
sed -i 's/bind 127.0.0.1/#bind 127.0.0.1/' /usr/local/redis-5.0.1/redis.conf
sed -i 's/protected-mode yes/protected-mode no/' /usr/local/redis-5.0.1/redis.conf
sed -i 's/port 6379/port 6377/' /usr/local/redis-5.0.1/redis.conf
```


5.启动，两种方式，一般通过后台进程启动
1.直接启动redis（只在安装完后测试用一下，一般都是后台启动）
cd /usr/local/redis-5.0.1/src
./redis-server
该启动方式需要一直打开窗口，ctrl + c退出。

2.后台进程方式启动,指定文件启动
./redis-server ../redis.conf

测试连接：
```
[root@localhost redis-5.0.1]# cd /usr/local/redis-5.0.1/src
[root@localhost src]# redis-cli
127.0.0.1:6379> 
[root@localhost src]# redis-cli -p 6377        //修改端口后需这样进入 
127.0.0.1:6377> 
```

设置软连接，让redis指令可以在任意目录下直接使用
```
ln -s /usr/local/redis-5.0.1/src/redis-cli /usr/bin/redis
[root@localhost ~]# redis
127.0.0.1:6379> 
[root@localhost ~]# redis -p 6377        //修改端口后需这样进入
127.0.0.1:6377> 
```
或者将redis-cli拷贝到/usr/bin/下，让redis-cli指令可以在任意目录下直接使用
cp /usr/local/redis-5.0.1/src/redis-cli /usr/local/bin


设置密码
```
设置redis密码，一般也不设置吧，不清楚，反正我们公司总没有设置，具体看要求吧
cd /usr/local/redis-5.0.1/
打开redis.conf文件，搜索requirepass关键字，在#requirepass foobared下面添加一行
requirepass	123456         #设置redis密码为123456

保存退出并重启redis

通过客户端连接的话必须加上密码参数才能正常连接： 
[root@localhost redis-5.0.1]# redis -a 123456
```


redis基本检测
```
cd /usr/local/bin
[root@localhost bin]# ./redis-benchmark       //默认6379端口
[root@localhost bin]# ./redis-benchmark -p 6377       //指定redis端口为6377
```

















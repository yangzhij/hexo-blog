---
title: rabbitMQ单机
date: 2018-12-14 10:24:33
tags:
---
RabbitMQ 是面向交换机投递消息的。消费者消费消息是面向消息队列的。
消息生产者（往 RabbitMQ 发消息的程序）把消息给交换机, 交换机根据调度策略（路由键）再把消息再给队列。
几个重要的概念：
虚拟主机，一个虚拟主机持有一组交换机、队列和绑定。每一个RabbitMQ服务器都有一个默认的虚拟主机"/"
交换机，接收消息并且转发到绑定的队列，交换机不存储消息。如果没有队列绑定到交换机的话，它会直接丢弃掉生产者发送过来的消息。
队列，每个消息都会被投入到一个或多个队列。
绑定（binding），作用是交换机需要和队列相绑定。不绑定无法进行消息转发。

如何保证消息的不丢失，三个地方做到持久化。
Exchange需要持久化。Queue需要持久化。Message需要持久化。

rabbitmq 安装  //172.20.100.207
```
//安装
yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget
wget http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el6.x86_64.rpm
rpm -ivh erlang-19.0.4-1.el6.x86_64.rpm
wget https://dl.bintray.com/rabbitmq/rpm/rabbitmq-server/v3.6.x/el/6/noarch/rabbitmq-server-3.6.15-1.el6.noarch.rpm
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
yum -y makecache
yum -y install socat
rpm -ivh rabbitmq-server-3.6.15-1.el6.noarch.rpm

//rabbitmq本身基本上不需要配置就能满足大多数需求，这不是我说的，是官网说的。

//系统调优
文件末尾添加下面4行
vi /etc/security/limits.conf
* soft nofile 102400
* hard nofile 102400

* soft nproc 102400
* hard nproc 102400

//文件末未添加
vi /etc/sysctl.conf
fs.file-max=102400   #保存退出
sysctl -p  生效配置
//可以打开的最大文件数量及用户最大可用的进程数
ulimit -n 65000
ulimit -u 65000

//启动并查看状态
systemctl start rabbitmq-server.service
systemctl status rabbitmq-server.service
chkconfig rabbitmq-server on
```
//开启网页管理插件
```
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server.service
```
//默认网页是不允许访问的，需要增加一个用户修改一下权限，代码如下：
```
rabbitmqctl add_user admin admin123  //添加用户
rabbitmqctl set_permissions -p / admin ".*"  ".*"  ".*"           //添加权限
rabbitmqctl set_user_tags admin administrator                  //修改用户角色
rabbitmqctl delete_user guest            //删除guest用户
```
浏览器访问：
http://172.20.100.207:15672


```
//查看mq状态
rabbitmqctl status
```

```
程序配置文件配置mq接口：172.20.100.207:5672
```

//防火墙:开通防火墙上Web访问端口
```
firewall-cmd --permanent --add-port=15672/tcp
firewall-cmd –-reload
```

批量添加mq用户主机  见github


















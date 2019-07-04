---
title: rabbitMQ集群
date: 2019-02-06 11:43:16
password: passw0rd
tags:
---
rabbitmq：消息队列。作为消息中间件，一般以集群方式部署，主要提供消息的接受和发送，实现各服务之间的消息异步。

原理：
RabbitMQ底层是通过Erlang架构来实现的。RabbitMQ底层是通过Erlang架构来实现的，所以rabbitmqctl会启动Erlang节点，并基于Erlang节点来使用Erlang系统连接RabbitMQ节点，在连接过程中需要正确的Erlang Cookie和节点名称，
Erlang节点通过交换Erlang Cookie以获得认证。
所以部署rabbitmq分布式集群时要先安装erlang，并把其中一个服务的cookie复制到另外的节点。

rabbitmq集群中，各个rabbitmq为对等节点，即每个节点均提供给客户端连接，进行消息的接收和发送。
节点分为内存节点和磁盘节点，一般的，为了防止机器重启后的消息消失，均应建立为磁盘节点；

RabbitMQ的Cluster集群模式一般分为两种，普通模式和镜像模式。

1.必备工具安装
```
yum -y install vim wget ntp lrzsz
```
2.确定3台主机的时间同步
```
yum -y install chrony
timedatectl set-timezone Asia/Shanghai
systemctl enable chronyd.service
systemctl start chronyd.service
hwclock --systohc
```
3.配置3台机器的/etc/hosts文件: 在/etc/hosts中，绑定三台机器的ip和主机名（erlang是通过主机名来连接服务，必须保证各个主机名之间可以ping通。如果主机名ping不通，rabbitmq服务启动会失败。）
```
vim /etc/hosts 添加（注意中间空格为tab符号）
192.168.0.101 rabbitmq-1
192.168.0.102 rabbitmq-2
192.168.0.103 rabbitmq-3
scp /etc/hosts root@192.168.0.102:/etc/hosts      拷贝到其他机器
scp /etc/hosts root@192.168.0.103:/etc/hosts
```
4.分别在3台上面执行安装
```
yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget
wget http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el6.x86_64.rpm
rpm -ivh erlang-19.0.4-1.el6.x86_64.rpm
wget https://dl.bintray.com/rabbitmq/rabbitmq-server-rpm/rabbitmq-server-3.6.12-1.el6.noarch.rpm
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
yum -y makecache
yum -y install socat
rpm -ivh rabbitmq-server-3.6.12-1.el6.noarch.rpm
```
5.**erlang.cookie是erlang实现分布式的必要文件，erlang分布式的每个节点上要保持相同的。**同时要保证cookie文件的权限是400,不然节点之间就无法通信。打开rabbitmq-1服务器的/var/lib/rabbitmq/.erlang.cookie中的内容,复制到其他节点上。
```
cd /var/lib/rabbitmq/
scp .erlang.cookie root@192.168.0.102:/var/lib/rabbitmq/
```
6.系统调优
```
文件末尾添加下面4行
vi /etc/security/limits.conf
* soft nofile 102400
* hard nofile 102400

* soft nproc 102400
* hard nproc 102400

//文件末未添加
vi /etc/sysctl.conf
fs.file-max=102400

//可以打开的最大文件数量及用户最大可用的进程数
ulimit -n 65000
ulimit -u 65000
```
7.分别启动
```
//启动并查看状态
systemctl start rabbitmq-server.service
systemctl status rabbitmq-server.service

//开启网页管理插件
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server.service
```

8.集群节点添加（添加新节点）
将rabbit@rabbitmq-1作为集群主节点，在其他节点上分别执行如下命令，以加入集群中.
```
rabbitmqctl stop_app                            # 仅停止应用
rabbitmqctl reset                                # 重置应用
rabbitmqctl join_cluster rabbit@rabbitmq-1           # 加入rabbit@rabbitmq-1集群
rabbitmqctl start_app                            # 开启应用
rabbitmqctl cluster_status                        # 都查看集群状态
```

9.账号管理
```
rabbitmqctl add_user admin admin                # 添加账号
rabbitmqctl set_user_tags admin administrator    # 添加权限
rabbitmqctl delete_user guest                    # 删除用户
rabbitmqctl change_password 用户名 密码            # 修改用户的密码
rabbitmqctl list_users                            # 查看当前用户列表

//vhost管理
rabbitmqctl add_vhost vhostname        # 创建vhost
rabbitmqctl delete_vhost vhostname     # 删除vhost
rabbitmqctl list_vhosts                # 查看所有虚拟主机信息
rabbitmqctl set_permissions -p v_host user  ".*"  ".*"  ".*"   ##绑定权限，并且具备读写的权限
rabbitmqctl list_queues                # 显示所有队列
```

10.分别在3台主机上查看连接的状态
rabbitmqctl cluster_status

–第一行是集群中的节点成员，disc表示这些都是磁盘节点。
–第二行是正在运行的节点成员

11.节点退出集群(从集群中移除节点)
```
假设要把rabbitmq-2退出集群，在rabbitmq2上执行
#rabbitmqctl stop_app
#rabbitmqctl reset
#rabbitmqctl start_app

然后在集群主节点上执行
# rabbitmqctl forget_cluster_rabbitmq rabbit@rabbitmq-2
```
12.可以删除默认guest用户
```
rabbitmqctl delete_user guest
```

haproxy 对mq进行负载
keepalive    提供统一入口，防止单点故障

**安装keepalived，提供一个入口**
```
yum -y install keepalived 
systemctl enable keepalived
systemctl start keepalived
```
分别编辑三台的配置文件vim /etc/keepalived/keepalived.conf，内容如下：priority 分别为101,102,103
```
vrrp_script chk_haproxy {
    script "/usr/sbin/pidof haproxy"
    interval 2
}
vrrp_instance VI_1 {
    interface eth0
    state BACKUP
    nopreempt
    priority 101
    virtual_router_id 10
    authentication {
        auth_type PASS
        auth_pass password
    }
    virtual_ipaddress {
        172.20.100.30
    }
    track_script {
        chk_haproxy
    }
    notify_master /etc/keepalived/check_haproxy.sh
}
```

//编辑检测脚本，为了防止haproxy服务关闭导致keepalived不自动切换
```
vim /etc/keepalived/check_haproxy.sh
#!/bin/bash
if [ $(ps -C haproxy --no-header | wc -l) -eq 0 ]; then
       systemctl start haproxy
fi
sleep 2
if [ $(ps -C haproxy --no-header | wc -l) -eq 0 ]; then
       systemctl stop keepalived
fi
```

//重启启动
systemctl restart keepalived

**haproxy搭建**
//安装
yum -y install haproxy

//配置haproxy
```
#vim /etc/haproxy/haproxy.cfg       //在末尾添加以下内容

#---------------------------------------------------------------------
listen rabbitmq_admin
    bind 0.0.0.0:15673
    mode tcp
    balance roundrobin
    option tcpka
    option tcplog
    timeout client 3h
    timeout server 3h
    timeout connect 3h
    server mqnode1 192.168.0.101:15672
    server mqnode2 192.168.0.102:15672
    server mqnode3 192.168.0.103:15672
	
	
listen rabbitmq_cluster 0.0.0.0:5673
    mode tcp

    balance roundrobin
    option tcpka
    option tcplog
    timeout client 3h
    timeout server 3h
    timeout connect 3h

    server   mqnode1 192.168.0.101:5672 check inter 5s rise 2 fall 3
    server   mqnode2 192.168.0.102:5672 check inter 5s rise 2 fall 3
    server   mqnode3 192.168.0.103:5672 check inter 5s rise 2 fall 3

```

//启动
```
systemctl start haproxy
```

**镜像模式**：把需要的队列做成镜像队列，存在于多个节点，属于RabbitMQ的HA方案，在任意节点上执行
```
rabbitmqctl set_policy ha-all "hello" '{"ha-mode":"all"}'
```
hello 是同步的队列名，可以用正则表达式匹配；
{“ha-mode”:”all”} 表示同步给所有























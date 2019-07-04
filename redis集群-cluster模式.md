---
title: redis集群-cluster模式
date: 2018-12-12 08:18:08
tags:
---

之前应开发要求，真真切切的搭建了一次redis集群。特根据查阅相关资源的内容以及具体实现做一个笔记。

 
集群一、 6台服务器，每台起一个7000端口 

集群二、 3台云服务器，每台起3个端口（7000,7001,7002，7003,7004,7005，7006,7007,7008）
（3台云服务器，每台启动3个redis节点，一共9个。并让它们按照Cluster模式工作起来。每台开启了3个redis监听实例，就等于3个redis了但在生产环境中并不建议在一台机器上部署多个Redis节点，因为这样会增加了多个Redis节点同时不可用的风险。）
 
 
Redis，非关系数据库，运行在内存拥有更快的读写。性能极高，数据类型丰富。redis是一种典型的no-sql 即非关系数据库 像python的字典一样 存储key-value键值对 工作在memory中。

作用：减轻db的压力

memcache和redis都工作在memory中，但memcache重启或者宕机memory中的数据就全没了，数据的一致性的不到保障. 但是redis不同,redis有相对的数据持久化的方案（rdb和aof） 

redis集群实现方案： 

1、豌豆荚开发的codis
2、redis官方的redis-cluster

redis-cluster是三个里性能最强大的 因为他使用去中心化的思想。
codis使用的也是proxy思路。是这两种之间的一个中间级 而且支持redis命令是最多的 有图形化GUI管理和监控工具，运维友好。


Redis 集群是一个可以在多个 Redis 节点之间进行数据共享的设施
Redis集群提供了以下两个好处：
将数据自动切分split到多个节点的能力。
当集群中的一部分节点失效或者无法进行通讯时，仍然可以继续处理命令请求的能力。


redis-cluster：
所有的redis节点彼此互联
客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可。


什么时候整个集群不可用？
如果集群任意master挂掉,且当前master没有slave.集群进入fail状态
如果集群超过半数以上master挂掉，无论是否有slave，集群进入fail状态.


------------------------------具体实现


检查是否安装Java
java -version
```
vi /etc/profile
export JAVA_HOME=/usr/local/java/jdk1.8.0_171
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile
```
安装redis
```
cd /usr/local/src
yum -y install gcc-c++
wget http://download.redis.io/releases/redis-4.0.8.tar.gz
tar zxvf redis-4.0.8.tar.gz -C /usr/local/
cd /usr/local/redis-4.0.8
mkdir redis_cluster
cd redis_cluster
mkdir 7000 7001 7002   （在第一台服务器创建，第二台7003,7004,7005，第三台7006，7007，7008）
```
#把redis文件分别复制到创建的目录下，然后更改配置文件中的内容
cp /usr/local/redis-4.0.8/redis.conf /usr/local/redis-4.0.8/redis_cluster/7000
cp /usr/local/redis-4.0.8/redis.conf /usr/local/redis-4.0.8/redis_cluster/7001
cp /usr/local/redis-4.0.8/redis.conf /usr/local/redis-4.0.8/redis_cluster/7002

```
daemonize yes
port 7000 （对应文件夹）
cluster-enabled yes
bind 0.0.0.0
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```


通过端口启动多个redis实例...启动第一台上的三个redis节点
```
/usr/local/redis-4.0.8/src/redis-server /usr/local/redis-4.0.8/redis_cluster/7000/redis.conf
/usr/local/redis-4.0.8/src/redis-server /usr/local/redis-4.0.8/redis_cluster/7001/redis.conf
/usr/local/redis-4.0.8/src/redis-server /usr/local/redis-4.0.8/redis_cluster/7002/redis.conf
```


下载ruby-2.3.7.tar.gz
```
tar zxvf ruby-2.3.7.tar.gz -C /usr/loacl
cd ruby-2.3.7
./configure
make && make install
 
ln -s /usr/local/bin/ruby /usr/bin/ruby
ln -s /usr/local/bin/gem /usr/bin/gem
```
安装gem redis接口
gem install redis


------报错1
```
[root@izj6cac2xjnladms506dsxz src]# gem install redis
ERROR:  Loading command: install (LoadError)
        cannot load such file -- zlib
ERROR:  While executing gem ... (NoMethodError)
    undefined method `invoke_with_build_args' for nil:NilClass
#解决
yum -y install gcc zlib-devel
cd /usr/local/ruby-2.3.7/ext/zlib
ruby ./extconf.rb
make &&make install
```

-------报错2
```
[root@izj6cac2xjnladms506dsxz zlib]# gem install redis
ERROR:  While executing gem ... (Gem::Exception)
    Unable to require openssl, install OpenSSL and rebuild ruby (preferred) or use non-HTTPS sources
#解决：		
cd ruby-2.3.7/
./configure  --with-openssl-dir=/usr/local/ssl
make &&make install
```


执行创建集群命令（也就是把集群中所有节点的ip:port串联起来）
```
/usr/local/redis-4.0.8/src/redis-trib.rb create --replicas 1 47.244.31.250:7000 47.244.31.250:7001 47.244.31.250:7002 47.244.7.22:7003 47.244.7.22:7004 47.244.7.22:7005 47.90.122.138:7006 47.90.122.138:7007 47.90.122.138:7008
```

检查集群状态
./redis-trib.rb check 172.31.245.103:7002

 
./redis-cli -c -h 47.90.122.138 -p 7000
 
 
集群创建成功登陆任意redis结点查询集群中的节点情况。(随便一个ip对应的端口都可以登陆)
客户端以集群方式登陆：
[root@izj6cac2xjnladms506dsuz src]# ./redis-cli -c -h 47.244.31.250 -p 7000
172.31.245.103:7000> cluster nodes

 
其中-c表示以集群方式连接redis，-h指定ip地址，-p指定端口号
cluster nodes 查询集群结点信息
cluster info 查询集群状态信息


#单个端口启动不了，然后进入配置文件注释掉这两行
slave-serve-stale-data yes
slave-read-only yes
 
 
使用：
```
127.0.0.1:7000>set name aa1

127.0.0.1:7000>get name               (去哪个redis里取都可以)
```
 
 
数据分片： 哈希槽，共16384个，集群中的主节点各负责一部分。这种结构容易添加或删除节点。

//添加或删除节点
添加节点： 从原来的节点上各取一部分槽到新节点上
删除节点： 把要移除节点中的槽移到其他节点上，然后在集群中移除A即可。

从一个节点将哈希槽移到另一节点，不会停止服务，所以添加、删除、改变某节点的哈希槽数量，不会造成集群不可用。

6379  客户端通讯端口
16379  集群总线端口

//redis集群不可用的情况
1.集群半数以上的master挂掉，会造成集群不可用
2.集群任意一个master节点挂掉，且当前master没有slave，会造成redis集群不可用


以下内容参考：https://blog.csdn.net/it_hejinrong/article/details/79205528

//添加一个master主节点：
```
1.安装好新的redis节点，然后修改配置，启动该节点

2.执行，前者是新加的节点     后者是原集群的myself节点
./redis-trib.rb add-node 10.0.0.7:7007 10.0.0.1:7001

执行完成后，可以通过cluster nodes 查看下

3.添加完主节点需要对主节点进行hash槽分配，这样该主节才可以存储数据。

redis集群有16384个槽，集群中的每个master结点分配一些槽，通过查看集群结点可以看到槽占用情况。

第一步：连接上集群 
./redis-trib.rb reshard 10.0.0.1:7001  （连接集群中任意一个可用节点都行）

第二步：输入要分配的槽数量
How many slots do you want to move (from 1 to 16384)? 1000             输入1000表示要分配1000个槽


第三步：输入接收槽的结点id

准备给7007分配槽，通过cluster nodes查看7007结点id为5d6c61ecff23bff3b0fb01a86c66d882f2d402a0

What is the receiving node ID? 5d6c61ecff23bff3b0fb01a86c66d882f2d402a0

第四步：输入源结点id 
输入源结点id，槽将从源结点中拿，分配后的槽在源结点中就不存在了。 
输入all表示从所有源结点中获取槽。 
输入done取消分配。

Source node #1:all

第五步：输入yes开始移动槽到目标结点id

Do you want to proceed with the proposed reshard plan (yes/no)? 
```


//添加一个slave从节点
```
集群创建成功后可以向集群中添加节点，下面是添加一个slave从节点

添加7008从结点，将7008作为7007的从节点。
1.安装好新的redis节点，然后修改配置，启动该节点

2.
./redis-trib.rb add-node --slave --master-id 主节点id 添加节点的ip和端口 集群中已存在节点ip和端口

[root@localhost redis-cluster]# ./redis-trib.rb add-node --slave --master-id 5d6c61ecff23bff3b0fb01a86c66d882f2d402a0 192.168.37.131:7008 192.168.37.131:7001
```


//删除集群节点中的某一个节点
```
想要删除集群节点中的某一个节点，需要严格执行2步：
1、 将这个节点上的所有插槽转移到其他节点上；
a) 假设我们想要删除6380这个节点
b) 执行脚本：./redis-trib.rb reshard 192.168.56.102:6380
c) 选择需要转移的插槽的数量，因为3380有5128个，所以转移5128个
d) 输入转移的节点的id，我们转移到6382节点：82ed0d63cfa6d19956dca833930977a87d6ddf7
e) 输入插槽来源id，也就是6380的id
f) 输入done，开始转移


2、 使用redis-trib.rb删除节点
./redis-trib.rb del-node 192.168.56.102:6380 4a9b8886ba5261e82597f5590fcdb49ea47c4c6c
```
 

 


 
 




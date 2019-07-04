---
title: ELK部署
date: 2019-01-18 12:49:36
tags:
---

elasticsearch默认安装后设置的内存是1GB，对于任何一个现实业务来说，这个设置都太小了。如果你正在使用这个默认堆内存配置，你的集群配置可能会很快发生问题。
#最简单的一个方法就是指定ES_HEAP_SIZE环境变量。建议设置内存的一半，但不超过32G     
```
//命令行直接设置 
export ES_HEAP_SIZE=16g
//或者如下
vi /etc/sysconfig/elasticsearch 
ES_HEAP_SIZE=16g #设置内存，建议设置系统内存的一半
```
//关闭防火墙
```
setenforce 0
systemctl stop firewalld.service
```
//安装相关软件
```
先检查有没有java环境变量,配置好后进行如下操作
yum -y install screen
yum -y install wget
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.3.tar.gz
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.2.3-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.2.3.tar.gz
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.2.3-linux-x86_64.tar.gz
tar zxvf elasticsearch-6.2.3.tar.gz
tar zxvf kibana-6.2.3-linux-x86_64.tar.gz
mv kibana-6.2.3-linux-x86_64 kibana-6.2.3
tar zxvf logstash-6.2.3.tar.gz
tar zxvf filebeat-6.2.3-linux-x86_64.tar.gz
```
//创建启动启动elk的用户
```
useradd  -m elastic
chown elastic.elastic -R /usr/local/elk/elasticsearch-6.2.3
```
//更改配置
```
sed -i 's/#cluster.name: */cluster.name: aaa/' /usr/local/elk/elasticsearch-6.2.3/config/elasticsearch.yml
sed -i 's/#node.name: */node.name: $hostname/' /usr/local/elk/elasticsearch-6.2.3/config/elasticsearch.yml
sed -i 's/#node.master: */node.master: true/' /usr/local/elk/elasticsearch-6.2.3/config/elasticsearch.yml
sed -i 's/#path.data: */path.data: /data/es-data/' /usr/local/elk/elasticsearch-6.2.3/config/elasticsearch.yml
sed -i 's/#path.logs: */path.logs: /var/log/es-log/' /usr/local/elk/elasticsearch-6.2.3/config/elasticsearch.yml
sed -i 's/#bootstrap.memory_lock: */bootstrap.memory_lock: true/' /usr/local/elk/elasticsearch-6.2.3/config/elasticsearch.yml
sed -i 's/#network.host: */network.host: $ip/' /usr/local/elk/elasticsearch-6.2.3/config/elasticsearch.yml
sed -i 's/#http.port: */http.port: 9202/' /usr/local/elk/elasticsearch-6.2.3/config/elasticsearch.yml

/* 
cluster.name: my-elk  ##elk集群名称，随意。注意顶格，冒号后空一格，下同
node.name: host1      ##节点1名称，注意解析问题
path.data: /var/lib/elasticsearch/  ##数据存储路径，当然可以自定义，如下图，elasticsearch用户自动创建，不用手动创建
bootstrap.mlockall: true    ##锁定内存
network.host: 172.25.0.1    ##本节点ip
http.port: 9200             ##apache端口 
*/
```
//创建es所需要的数据目录和日志目录，设置属性
```
mkdir -p /data/es-data
mkdir -p /var/log/es-log
chown -R elastic.elastic /data/es-data/
chown -R elastic.elastic /var/log/es-log/
```
//升级系统相关
```
echo "vm.max_map_count=655360" >>/etc/sysctl.conf
sysctl -p

echo "elastic hard nofile 65536" >>/etc/security/limits.conf
echo "elastic soft nofile 65536" >>/etc/security/limits.conf
```
//修改kibana配置文件
```
mv kibana-6.2.3-linux-x86_64 kibana-6.2.3
cd /usr/local/elk/kibana-6.2.3/config
sed -i 's/#server.port: 5601/server.port: 5601/' kibana.yml 
sed -i 's/#server.host: */server.host: $ip/' kibana.yml
sed -i 's/#elasticsearch.url/elasticsearch.url/' kibana.yml 

#server.port: 5601
#server.host: "10.0.0.1"
#elasticsearch.url: "http://10.0.0.1:9200" 
```

//设置kibana用service启动
```
vi /etc/systemd/system/kibana.service
[Service]
ExecStart=/usr/local/elk/kibana-6.2.3/bin/kibana

[Install]
WantedBy=multi-user.target

systemctl enable kibana.service
systemctl start kibana.service
systemctl status kibana.service
```

//es和logstash启动
```
screen -S elastic
su - elastic
cd /usr/local/elk/elasticsearch-6.2.3/bin
./elasticsearch
ctrl+a+d  退出
su -S logstash
cd /usr/local/elk/logstash-6.2.3/bin
./logstash -f ../conf/logstash.conf 
```

//查看集群状态：
```
curl -XGET 'http://172.20.100.219:9200/_cat/nodes'

curl -XGET 'http://172.20.100.219:9200/_cat/health?v'

curl -XGET 'http://172.20.100.219:9200/_cluster/state/nodes?pretty'
```

//索引相关命令
```
创建索引
curl -XPUT 'localhost:9200/two-cattle-service-2018.10.28'


清除索引：
curl -XDELETE 172.20.100.219:9200/two-cattle-service-2018.10.28


查看索引：
curl -XGET 'http://172.20.100.219:9200/_cat/indices/?v'
```

//设置nginx用户认证
```
yum -y install epel-release
yum -y install nginx httpd-tools
删除或注释掉文件/etc/nginx/nginx.conf中的一段server{}

cd /etc/nginx/conf.d/
vim kibana.conf          #输入以下内容

server {
    listen 80;
    #server_name 172.20.100.219;
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.users;
    location / {
            proxy_pass http://172.20.100.219:5601;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
    }
}
~
保存退出


配置用户文件
touch /etc/nginx/htpasswd.users      

配置登录验证
yum install -y httpd-tools
htpasswd -bc /etc/nginx/htpasswd.users admin admin123

查看文件，密码会加密
cat /usr/local/nginx/conf/htpasswd.users  

cd /etc/nginx/conf.d/
nginx -t      --检测

systemctl start nginx
systemctl enable nginx


浏览器访问 http://172.20.100.219  登陆即可     （此为内网IP，必要的话，映射一个外网ip或域名即可）

然后http://172.20.100.219:5601 也可以，  这个不需要密码登入

```


集群自动选举：

elasticsearch集群一旦建立起来以后，会选举出一个master，其他都为slave节点。
但是具体操作的时候，每个节点都提供写和读的操作。就是说，你不论往哪个节点中做写操作，这个数据也会分配到集群上的所有节点中。

这里有某个节点挂掉的情况，如果是slave节点挂掉了，那么首先关心，数据会不会丢呢？不会。如果你开启了replicate，那么这个数据一定在别的机器上是有备份的。
别的节点上的备份分片会自动升格为这份分片数据的主分片。这里要注意的是这里会有一小段时间的yellow状态时间。

如果是主节点挂掉怎么办呢？当从节点们发现和主节点连接不上了，那么他们会自己决定再选举出一个节点为主节点。
但是这里有个脑裂的问题，假设有5台机器，3台在一个机房，2台在另一个机房，当两个机房之间的联系断了之后，每个机房的节点会自己聚会，推举出一个主节点。
这个时候就有两个主节点存在了，当机房之间的联系恢复了之后，这个时候就会出现数据冲突了。
解决的办法就是设置参数：

discovery.zen.minimum_master_nodes
为3(超过一半的节点数)，那么当两个机房的连接断了之后，就会以大于等于3的机房的master为主，另外一个机房的节点就停止服务了。


//集群
```
ABC三台安装elk集群  (不要安装x-pack，收费，只能免费使用一个月，一个月后就报错;   使用x-pack后查集群信息就查不出来了;   )

A -es,logstash,kibana
B,C -es


安装x-pack插件：    在A上安装配置密码即可（es,kibana）。  然后分别在BC上安装es的x-pack，不用配置密码。 然后才能在kibana启动。


kibanna
cd /usr/local/elk/kibana-6.2.3/bin
./kibana-plugin install x-pack


es  (安装x-pack的时候，先把BC两台es停掉，然后安装)
cd /usr/local/elk/elasticsearch-6.2.3/bin
./elasticsearch-plugin install x-pack
chown elastic.elastic /usr/local/elk/elasticsearch-6.2.3/config/elasticsearch.keystore
./x-pack/setup-passwords interactive


cd /usr/local/elk/
wget https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-6.2.3.zip
cd /usr/local/elk/elasticsearch-6.2.3/bin
./elasticsearch-plugin install file:///usr/local/elk/x-pack-6.2.3.zip

```

//根据索引删除elk历史日志
```
//编写脚本
#!/bin/bash
aa=`cat server.conf`
ip=172.20.100.219
port=9200
date="2018.10.29"
date1="2018.10.30"
for i in $aa
do
    curl -XDELETE $ip:$port/$i-$date
    curl -XDELETE $ip:$port/$i-$date1    
done

//同目录下编写文件，输入日志索引名称
vi server.conf    把索引名称写入文件

account-service
activity-service

```

























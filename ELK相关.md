---
title: ELK相关
date: 2019-01-18 13:59:07
tags:
---
//之前做es的配置，其余节点只需改node.name即可，剩下的都一样
```
cluster.name: my-elk
node.name: node-2
path.data: /data/es-data
path.logs: /var/log/es-log
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: ["172.20.100.219","172.20.100.135","172.20.100.136"]
```


//es配置参考
```
[root@host1 ~]# vim /etc/elasticsearch/elasticsearch.yml 
cluster.name: my-elk
node.name: node-1
node.master: true    ##做master角色
node.data: false     ##不做数据节点
path.data: /var/data/es-data
bootstrap.mlockall: true
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: ["172.20.100.219","172.20.100.135","172.20.100.136"]


[root@host2 ~]# vim /etc/elasticsearch/elasticsearch.yml 

cluster.name: my-elk
node.name: node-2
node.master: false   #不做master节点
node.data: true      #只做数据节点，只负责存储数据，后期提供存储和查询服务
path.data: /var/data/es-data
path.logs: /var/log/es-log
bootstrap.mlockall: true      //添加此选项，防止在内存不够用的时候，elasticsearch的内存被交换至交换区，导致性能骤降。保证了节点加入集群后的稳定性。
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: ["172.20.100.219","172.20.100.135","172.20.100.136"]



[root@host3 ~]# vim /etc/elasticsearch/elasticsearch.yml
cluster.name: my-elk
node.name: node-3
node.master: false      ##表示host3本节点不充当，精选master角色
node.data: true         ##表示host3节点充当数据节点
path.data: /var/data/es-data
bootstrap.mlockall: true
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: ["172.20.100.219","172.20.100.135","172.20.100.136"]
```

ELK集群
es 3台，其中一台+kibana
logstash+redis单独一台
filebeat 每台应用服务器
依次启动es,kibana,logstash,filebeat
使用filebeat的原因为logstash需要依赖java环境，对cpu和内存消耗都比较大。filebeat应运而生，对cpu和内存基本没有消耗。
使用redis 的原因是当多个clinet同时写入到logstash或者elasticsearch 时候，有io瓶颈，所以选择了redis ,当然可以使用kafka，rabbitmq等消息中间件。
```
filebeat配置
cd /usr/local/src/filebeat
vi filebeat.yml
filebeat.prospectors:
- type: log
  #设置为true使该配置生效
  enabled: true
  paths:
   - /home/loguser/logs/fish1/daily/*.log
  fields:
   log_source: 172.20.100.72
   log_index: goldcarp
- type: log
  enabled: true
  paths:
   - /home/loguser/logs/ebg/*.log
  fields:
   log_source: 172.20.100.72
   log_index: two-eight
  #下面三个属性配置根据[ 将java异常堆栈收集为一条消息,也可以logstash中配置
  multiline.pattern: '^\['
  multiline.negate: true
  multiline.match: after
  #该属性可以配置只收集error级别和warn级别的日志,如果有配置多行收集,一定要将这个配置放在多行的后面
  include_lines: ['ERROR','WARN','INFO']
  #该属性配置不收集DEBUG级别的日志,如果配置多行 这个配置也要放在多行的后面
  exclude_lines: ['^#']
output.logstash:
  #如果是多个logstash将下面三条属性打开
  #hosts: ["localhost:5044", "localhost:5045"]
  #loadbalance: true
  #worker: 3
  hosts: ["172.20.100.185:5044"]
  #这个index的值也可以自定义,用来修改@metadata[beat]值，默认是filebeat
  index: "filebeat"
  
  #如果是输入到redis，用下面这个
#output.redis:
  #hosts: ["192.168.147.190"]
  #password: "6lapp"
  #key: "log_file"
  #db: 0
  #timeout: 5  
  
#FileBeat控制台启动方式
cd /usr/local/src/filebeat
./filebeat -e -c filebeat.yml -d "publish"  


#lohstash配置
vi logstash.conf
input {
    beats {
        port => 5044

    }
}

#如果是输出到redis，用下面这个
#input { 
#    redis {
#       port => "6379"
#       host => "192.168.147.190" 
#       password => "6lapp"
#       data_type => "list" 
#}
#}

filter{

          mutate { 
		  #移除不想要的字段，有几个是去不掉的
            remove_field =>["@version","offset","beat","log","prospector","_id","tags","_type","_score"]

      }
		  date {
			match => ["timestamp", "dd/MM/yyyy:HH:mm:ss Z" ]
			remove_field => ["timestamp"]
		  }

}

output {
    elasticsearch {
    hosts => ["http://172.20.100.185:9200"]
        #如果是多台es,就如下配置
        #hosts => ["172.20.103.150:9200","172.20.103.151:9200","172.20.103.152:9200"]
    #默认使用[@metadata][beat]来区分不同的索引,也就是filebeat中配置的index字段
    #index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    #我使用自定义fields字段来区分不同的索引
        #输出结果是goldcarp-6.2.3-2019.05.06
        #index => "%{[fields][log_index]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
        index => "%{[fields][log_index]}-%{+YYYY.MM.dd}"
  }

}  

```




































---
title: zabbix监控rabbitmq
date: 2019-03-12 20:13:24
password: passw0rd
tags:
---
rabbitMQ监控
	1. 运行状态
		mq本身运行状态
		消息队列状态：有无数据
		erlang VM运行时间（没有用，写不写无所谓）
		
	2. 服务器性能
		流量 incoming    outcoming
		CPU   -->CPU system time  CPU在内核运行的时间
		Memory --> free memory 
		磁盘空间 /   Filesystems     Free disk space on /
		
	3. rabbitMQ性能
		消息与准备就绪的消息
		消费者个数和连接数
		总共读取磁盘数据大小
		写入磁盘数据总量

在客户端添加rabbitmq_status.py和rabbitmq_status.conf文件，文件在github上（https://github.com/yangzhij/zhifou下）。机器上文件路径为：
```
/etc/zabbix/rabbitmq_status.py
/etc/zabbix/zabbix_agentd.d/rabbitmq_status.conf
```
zabbix界面操作：
1.新建主机组，如rabbitMQ
2.添加主机并加入属组linux server及rabbitMQ
3.新建模板Template OS RabbitMQ
4.创建应用集RabbitMq Node、RabbitMq Overview、RabbitMq Queues
5.添加监控项，如下
![https://raw.githubusercontent.com/yangzhij/images2/master/zmq1.png](https://raw.githubusercontent.com/yangzhij/images2/master/zmq1.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/zmq2.png](https://raw.githubusercontent.com/yangzhij/images2/master/zmq2.png)

```
名称                           键值
rabbitmq.active             rabbitmq.active
RabbitMq Channels			rabbitmq[overview,channels]
RabbitMQ Connections		rabbitmq[overview,connections]
RabbitMQ Consumers	        rabbitmq[overview,consumers]	     
RabbitMQ Exchanges	        rabbitmq[overview,exchanges]	     
RabbitMQ Messages	        rabbitmq[overview,messages]	             
RabbitMQ Messages_ready	 	rabbitmq[overview,messages_ready]
RabbitMQ Server IO Read Bytes Rate	 	rabbitmq[server,io_read_bytes]	
RabbitMQ Server IO Write Bytes Rate	 	rabbitmq[server,io_write_bytes]
RabbitMQ Server Uptime	 	rabbitmq[server,uptime]
Rabbit Queue Status		     rabbitmq.queues
```

创建触发器：
```
名称                                        表达式
rabbitmq is down                   {Template OS RabbitMQ:rabbitmq.active.last(,30)}=0
rabbitmq message heap up	       {Template OS RabbitMQ:rabbitmq[overview,messages].last(0)}>9000
rabbitmq queues nodata	           {Template OS RabbitMQ:rabbitmq.queues.last(0)}=0
```
![https://raw.githubusercontent.com/yangzhij/images2/master/zmq3.png](https://raw.githubusercontent.com/yangzhij/images2/master/zmq3.png)

结束了别忘了重启zabbix-agent哟~~~~


grafana上的操作	
![https://raw.githubusercontent.com/yangzhij/images2/master/mq1.png](https://raw.githubusercontent.com/yangzhij/images2/master/mq1.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/mq2.png](https://raw.githubusercontent.com/yangzhij/images2/master/mq2.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/mq3.png](https://raw.githubusercontent.com/yangzhij/images2/master/mq3.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/mq4.png](https://raw.githubusercontent.com/yangzhij/images2/master/mq4.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/mq5.png](https://raw.githubusercontent.com/yangzhij/images2/master/mq5.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/mq5-1.png](https://raw.githubusercontent.com/yangzhij/images2/master/mq5-1.png)
---
title: Mongodb副本集
date: 2019-02-09 20:47:04
password: passw0rd
tags:
---
**replication set**： 副本集，多台服务器维护相同的数据副本，提高服务器的可用性.
最小的副本集也应该具备一个primary节点和两个secondary节点。两个节点的副本集不具备真正的故障转移能力。

**整个流程说明：**
 本次是在一台机器上启动了3个实例进行模拟测试，区别于端口。副本集完成之后，27017是primary，27018和27019为Secondary。在27017上插入数据，27018和27019都可以读到。 把27017停掉之后，27018会从Secondary变成primary。27019还是Secondary。在27018上插入数据，27019也可以读到。当27017重新起来后，会从原来的primary变成Secondary，数据也会同步的。

**注意：**
当副本集中所有的Secondary都宕机、或者副本集中只剩下一个节点，则该节点只会成为Secondary节点（即使是primary节点也只能降级为secondary节点），也就意味着整个集群此时只能进行读操作而不能进行写操作，当其他节点恢复之后，之前的primary节点仍然是primary节点。**默认情况下，Secondary是不提供服务的，即不能读和写。在特殊情况下需要读的话则需要：rs.slaveOk()，只对当前连接有效。**

**//下载**
```
cd /usr/local
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.0.4.tgz
tar xvf mongodb-linux-x86_64-rhel70-4.0.4.tgz
mv mongodb-linux-x86_64-rhel70-4.0.4 mongodb
```
**//创建目录**
```
mkdir -p /home/m17 /home/m18 /home/m19 /home/mlog
```
**//启动实例，且声明实例属于某复制集。  本次是在一台机器上启动了3个实例，区别于端口。实际生产中可以3台机器各自启动一个，端口可以都是27017。**
说明：启动MongoDB有2种方式，一是直接指定配置参数，二是指定配置文件。本次通过指定配置参数启动：
```
//启动
/usr/local/mongodb/bin/mongod --dbpath /home/m17 --logpath /home/mlog/m17.log --port 27017 --bind_ip 0.0.0.0 --fork --smallfiles --replSet rs2
/usr/local/mongodb/bin/mongod --dbpath /home/m18 --logpath /home/mlog/m18.log --port 27018 --bind_ip 0.0.0.0 --fork --smallfiles --replSet rs2
/usr/local/mongodb/bin/mongod --dbpath /home/m19 --logpath /home/mlog/m19.log --port 27019 --bind_ip 0.0.0.0 --fork --smallfiles --replSet rs2
//参数说明
--dbpath             #存放数据的路径
--logpath            #存放日志的路径
--bind_ip            #绑定ip。默认127.0.0.1，只能通过本地连接。
--fork               #后台启动
--smallfiles         #使用较小的默认数据文件大小。
--replSet rs2        #指定一个副本集名称rs2作为参数，所有主机都必须有相同的名称作为同一个副本集。
```
**//初始化**
```
cd /usr/local/mongodb/bin/
./mongo
> rs.initiate({"_id":"rs2","members":[
{"_id":0,
"host":"172.20.100.219:27017",
"priority":1
},
{"_id":1,
"host":"172.20.100.219:27018",
"priority":1
},
{"_id":2,
"host":"172.20.100.219:27019",
"priority":1
}
]})
```

**//主节点PRIMARY插入数据测试**
```
//测试1
rs2:PRIMARY> use test
rs2:PRIMARY> db.user.insert({uid:1,name:'lily'});
rs2:PRIMARY> db.user.find()

//测试2
rs2:PRIMARY> for(var i=0;i<10000;i++){db.test.insert({"name":"test"+i,"age":123})}
rs2:PRIMARY> db.test.count()
```

**//在Secondary上查看是否已经同步**
```
//测试1
rs2:SECONDARY> rs.slaveOk()
rs2:SECONDARY> db.user.find()
//测试2
rs2:SECONDARY> rs.slaveOk()
rs2:SECONDARY> db.test.count()
```


**增删节点，注意：增删节点都在PRIMARY节点执行**
//添加节点到副本集
```
rs2:PRIMARY> rs.add("172.20.100.219:27016")

rs2:PRIMARY> rs.status()  //查看副本集的状态

```

//从副本集移除某节点
```
rs2:PRIMARY> rs.remove("172.20.100.219:27016")
rs2:PRIMARY> rs.status()  //查看副本集的状态
```


**手动切换Primary节点到自己给定的节点**
```
rs2:PRIMARY> rs.conf()   #查看配置

"_id" : "rs2",       副本集名称
"version" : 7,       每改变一次集群的配置，副本集的version都会加1
...
...

members里面其中一个参数：
priority  ：优先集，因为最开始默认的都是1，只需要把给定的服务器的priority加到这些主机中最大的即可。

rs2:PRIMARY> rs.status() #查看状态

rs2:PRIMARY> cfg=rs.conf()


rs2:PRIMARY> cfg.members[1].priority=2  #修改priority         [1] 相当于python里面的下标，更改第几台主机的priority=2，从0开始算第一台
rs2:PRIMARY> rs.reconfig(cfg)           #重新加载配置文件，强制副本集进行一次选举，优先级高的成为Primary。在这期间整个集群的所有节点都是secondary。

rs2:PRIMARY> rs.status()      #查看状态，更改完后需要等一下才能生效，所以第一遍查看状态看不出来，多查看两次就出来效果了。
```




**//添加仲裁节点  "stateStr" : "ARBITER"**(4版本开始已经没有必要了)
副本集要求参与选举投票(vote)的节点数为奇数，为了实现 Automatic Failover（自动故障转移）引入另一类节点：仲裁者（arbiter），仲裁者只参与投票不拥有实际的数据，并且不提供任何服务，因此它对物理资源要求不严格。

**测试发现**，当整个副本集集群中达到50%的节点（包括仲裁节点）不可用的时候，剩下的节点只能成为secondary节点，整个集群只能读不能写。
当集群中有1个primary节点，1个secondary节点和1个arbiter节点，这时即使 primary节点挂了，剩下的secondary节点也会自动成为primary节点。
因为仲裁节点不复制数据，因此**利用仲裁节点可以实现最少的机器开销达到两个节点热备的效果。**

**如果要把之前的某一节点变成仲裁节点，则需要：**
1.先从副本集中移除该节点
2.杀掉该进程
3.删除该节点的数据目录和日志目录
4.重新创建好数据目录
5.重新启动然后添加到副本集rs.addArb("172.20.100.219:27019")


**//运行rs.status()发现某从节点statestr一直是STARTUP2，处理如下：**
1.先从副本集中移除该节点
2.杀掉该进程
3.删除该节点的数据目录和日志目录
4.重新创建好数据目录
5.重新启动然后添加到副本集



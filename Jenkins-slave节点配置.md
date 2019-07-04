---
title: Jenkins-slave节点配置
date: 2019-01-03 23:32:35
password: passw0rd
tags:
---

配置好slave节点后，在构建服务的时候，jenkins就会自动把项目分配给master或者slave节点来构建，在服务并发构建高的时候，可以节约时间，减少排队等待。所有的服务都可以在master上看到。添加服务的时候就直接在jenkins添加就可以了。
操作如下：
在slave机器上不需要安装jenkins，只需要安装java,git,maven,Gradle等项目需要的环境，安装路径和master保持一致，然后在master上按如下配置
![https://raw.githubusercontent.com/yangzhij/myself-images/master/slave1.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/slave1.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/slave2.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/slave2.png)


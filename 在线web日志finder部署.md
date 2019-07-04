---
title: 在线web日志finder部署
date: 2018-12-19 13:23:23
tags:
---
<font color="#FF0000">当时做的时候参考的这个：http://blog.51cto.com/superleedo/2135195</font> 
之前都是用elk的，因为主要是让开发人员看嘛，所以感觉也还行。后来在网上看了个Finder的最后效果，感觉不错，于是呢，就在测试平台搭建了一把，也简单也麻烦......现在测试环境用的就是这一款日志查询系统。
***Finder***：web方式的文件管理器，最主要的功能是web文件管理和日志文件的实时查看。
**部署：**
*安装java，并配置环境变量*
Master主机：
```
mkdir /home/rishi
cd /home/rishi
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.13/bin/apache-tomcat-9.0.13.tar.gz
tar zxf apache-tomcat-9.0.13.tar.gz
vi conf/server.xml   #更改端口为8186，随意不冲突即可
 
cd /home/rizhi/apache-tomcat-9.0.13/webapps/ROOT/
rm -rf *
wget http://www.finderweb.net/download.html/finder-web-2.4.9.war
jar -xvf finder-web-2.4.9.war && rm –rf finder-web-2.4.9.war
cd /home/rizhi/apache-tomcat-9.0.13/bin/
./ startup.sh
```
浏览器访问：
http://172.20.100.191:8186/finder
admin/1234

Slave：
客户端按照上诉过程安装好即可（直接打包Master上的服务，拷贝到相应的slave上启动就行），端口看具体情况，和本地冲突的话换一个就行。然后在master的web界面添加主机，更改相关配置即可。
![https://raw.githubusercontent.com/yangzhij/myself-images/master/finder1.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/finder1.png)
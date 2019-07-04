---
title: Tomcat安装及优化
date: 2019-06-20 16:36:28
tags:
---
~~很简单的，闲的没事就写一下喽
//安装前提
须有java及其环境变量, 安装配置见linux一些基础

//安装
```
cd /home
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.14/bin/apache-tomcat-9.0.14.tar.gz
tar zxvf apache-tomcat-9.0.14.tar.gz 
mv apache-tomcat-9.0.14 Jenkinstomcat          //可以把目录改个名字，便于区分
cd Jenkinstomcat/conf/
vi server.xml #有需要修改Tomcat端口,例如改为83端口   
sed -i 's/Connector port="8080"/Connector port="83"/' /home/Jenkinstomcat/conf/server.xml  
cd ../webapps/
项目是放到这个目录下的
cd ../bin/
./startup.sh
```

注：在一台机器上运行多个Tomcat实例，就把目录名改一下，server.xml里端口改一下，分别启动就ok了

//优化
Tomcat一般来讲做优化的话，除了机器本身优化(见linux基本优化),还有就是Tomcat容器内（conf目录下server.xml）针对吞吐量做优化，就是一些连接参数的优化.
```
<Connector port="83" protocol="HTTP/1.1"          
	URIEncoding="UTF-8"            
	minSpareThreads="25"    #表示即使没有人使用也开这么多空线程等待       
	maxSpareThreads="75"    #表示即使没有人使用也开这么多空线程等待      
	enableLookups="false"         #是否反查域名，取值为：true或false。为了提高处理能力，应设置为false  
	disableUploadTimeout="true"           
	connectionTimeout="20000"      #网络连接超时，单位：毫秒。 设置为0表示永不超时，这样设置有隐患的。   
	acceptCount="300"     #当同时连接的人数达到maxThreads时，还可以接收排队的连接，超过这个连接数则直接返回拒绝连接。       
	maxThreads="300"         #表示最多同时处理300个连接  
	maxProcessors="1000"     #最大连接线程数，即：并发处理的最大请求数，    
	minProcessors="5"        #最小空闲连接线程数，用于提高系统处理性能        
	useURIValidationHack="false"        #设置为false，可以减少它对一些url的不必要的检查从而减省开销  
	compression="on"      #开启压缩功能，HTTP 压缩可以大大提高浏览网站的速度      
	compressionMinSize="2048"       #启用压缩的输出内容大小，这里面默认为2KB   
	compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain"       #压缩类型   
	redirectPort="8443"    #走https协议的话，我们将会用到8443端口
/>      
```	
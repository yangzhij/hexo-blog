---
title: Maven私服搭建
date: 2018-12-28 00:14:47
tags:
---
*参考https://www.cnblogs.com/2YSP/p/9533506.html*

Maven：服务于java平台的自动化构建工具。构建工具演变：make--ant--Maven--Gradle
Maven采用了一种被称之为Project Object Model(POM)概念来管理项目。pom.xml对于maven工程是核心配置文件，所有的项目配置信息都在这个文件中进行配置。
作用：1.管理jar包	2.将项目拆分成若干个模块
Maven会将打包得到的文件复制到仓库中的指定位置target目录下。

maven仓库： 用来管理依赖，仓库保存的内容是maven工程。当 Maven 需要下载构件时，直接请求私服，私服上存在则下载到本地仓库；否则，私服请求外部的远程仓库，将构件下载到私服，再提供给本地仓库下载。
1.本地仓库：当前电脑上部署的仓库目录，为当前电脑上所有的maven工程服务
2.远程仓库
	a.中央仓库：架设在Internet上，为全世界所有的maven工程服务
	b.私服：搭建在局域网内，供局域网内的maven用户使用  （私服节省带宽，稳定性好，配置私服也可降低中央仓库的压力）



**搭建Maven私服的过程记录**
**一、安装java及maven**
1.java及环境变量，这个不说了
2.下载Maven包
```
cd /usr/local
wget https://mirrors.cnnic.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
tar -zxvf  apache-maven-3.5.4-bin.tar.gz
```
3.配置环境变量 vi /etc/profile
```
export MAVEN_HOME=/usr/local/apache-maven-3.5.4
export PATH="$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH"
```
**二、部署nexus**
1.下载
方法一：https://www.sonatype.com/download-oss-sonatype
![http://i2.bvimg.com/672478/8576db2083192fa6.png](http://i2.bvimg.com/672478/8576db2083192fa6.png)
方法二：wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz 
2.解压
```
cd /usr/local
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar zxvf latest-unix.tar.gz
//修改默认端口(8081)和允许所有主机访问 
vi nexus-3.14.0-04/etc/nexus-default.properties                 
application-port=8081
application-host=0.0.0.0
```
3.启动
```
cd /usr/local/nexus-3.14.0-04/bin
./nexus start
```
4.访问
http://ip:8081
点击右上角的sign in登录，输入账户admin，密码admin123即可登录成功。
![http://i2.bvimg.com/672478/9c319c384e46f26e.png](http://i2.bvimg.com/672478/9c319c384e46f26e.png)
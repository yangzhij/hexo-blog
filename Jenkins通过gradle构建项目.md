---
title: Jenkins通过gradle构建项目
date: 2018-12-22 16:22:48
password: passw0rd
tags:
---

之前都是用maven来构建项目的，这次研发给了个新服务，要通过gradle构建。刚开始还挺茫的，但网上资料挺多的……
过程如下：
先在jenkins所在的服务器安装好java、git和gradle
我刚开始安装的是gradle5版本的，可后来一直构建失败，研发那边用的是4版本的，本地测试可以。于是我又装了个低版本的。


Git安装：
```
yum –y install git
```
Gradle安装：
```
cd /usr/local
wget http://services.gradle.org/distributions/gradle-4.10.3-bin.zip 
unzip gradle-4.10.3-bin.zip
```
配置环境变量
```
Vi /etc/profile
export GRADLE_HOME=/usr/local/gradle-4.10.3
export PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin:${GRADLE_HOME}/bin:${JAVA_HOME}:${PATH}
source /etc/profile    #配置生效
```
配置jenkins:
系统管理---全局工具配置—Gradle：
gradle-4.10.3
/usr/local/gradle-4.10.3
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gradle1.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gradle1.png)

配置好后，开始创建任务：创建一个自由风格的任务
构建这里选择我们在jenkins中安装的gradle并在gradle全局工具配置中配置好的gradle，执行的tasks是clean build
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gradle2.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gradle2.png)

配置SSH发布
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gradle3.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gradle3.png)

执行命令如下
```
#!/bin/bash
pid=`ps -ef|grep pay|grep -v grep|awk '{print $2}'`
if [ -n "$pid" ]
then
    kill -9 $pid
fi
cd /home/wwwroot/
nohup /usr/local/java/jdk1.8.0_171/bin/java -jar pay-1.0.0.jar --spring.profiles.active=test >/dev/null 2>&1 &
if [ $? -eq 0 ]; then 
echo "pay server is Start....." 
fi 
exit 0

```

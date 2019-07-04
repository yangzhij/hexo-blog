---
title: Jenkins部署
date: 2018-12-16 12:08:08
tags:
---
由于架构体系的要求，已经部署过N多次jenkins了，部署完一次之后。后续不过都是基于现有的，直接把相关程序包拷贝到另一台新机器上即可。可能会提醒你有新版本了，这个更不更新无所谓的，看个人....
现在把部署过程做个记录......
先提一下jenkins是啥吧，害怕后面忘了...
jenkins,一个用于持续集成的工具，可以在研发把代码提交gitlab，svn等代码仓库后，通过jenkins拿到最新的代码后进行自动化编译，打包，上传服务器并部署。相对于原始的手动打包，上传等确实方便的多。简洁明了就一句话：持续集成即一个自动构建过程，包括自动编译，分发，部署和测试。
工作流程：在jenkins上点击某个按钮进行构建--jenkins收到命令后--从git或svn上把源码下载下来--根据设置的mvn命令进行打包--把打包好的jar包或者war包传到你的服务器上--启动服务
jenkins构建后操作，就是把通过maven命令打好包然后copy到服务器上。动态web工程打war包，java工程打jar包	

部署：  //172.20.100.176

<font color="#FF0000">**#jenkins属于java代码，需要java才能运行jenkins，So need install JDK+Tomcat**</font>

**1.查看java：**   java -version
**2.如果没有, oracle官网下载JDK,解压**
https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
```
cd /usr/local/
tar -zxvf jdk-8u171-linux-x64.tar.gz
```
**3.配置环境变量**
```
vi /etc/profile
export JAVA_HOME=/usr/local/java/jdk1.8.0_171
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile
```
**4.安装Tomcat+jenkins**
_#查看Tomcat版本： http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/_
```
cd /home
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.14/bin/apache-tomcat-9.0.14.tar.gz
tar zxvf apache-tomcat-9.0.14.tar.gz 
mv apache-tomcat-9.0.14 Jenkinstomcat
cd Jenkinstomcat/conf/
vi server.xml #有需要修改Tomcat端口,例如改为83端口   
sed -i 's/Connector port="8080"/Connector port="83"/' /home/Jenkinstomcat/conf/server.xml  
cd ../bin/
./startup.sh    #启动  172.20.100.176:83
cd ../webapps/
```
#将jenkins的war包放在Tomcat的webapps下，这里下载的是最新的,两个路径皆可
wget http://ftp.yz.yamagata-u.ac.jp/pub/misc/jenkins/war-stable/latest/jenkins.war 
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war 
_#具体版本可看http://ftp.yz.yamagata-u.ac.jp/pub/misc/jenkins/war-stable/_ 

**5.安装maven(直接解压即可,环境变量有没有无所谓吧，反正我一般总不配置)**
_#作用:负责java语言的编译和打包_
```
cd /usr/local
wget https://mirrors.cnnic.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
tar -zxvf  apache-maven-3.5.4-bin.tar.gz
```
-----当然了，安装完maven后是需要修改配置文件的，/usr/local/apache-maven-3.5.4/conf/settings.xml，这个里面需要修改maven私服的地址，以及登陆maven私服的用户名及密码。
```
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>admin123</password>
     </server>
```
![https://raw.githubusercontent.com/yangzhij/myself-images/master/maven2.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/maven2.png)
```
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <name>nexus</name>
      <url>http://172.20.100.202:8084/repository/maven-public/</url>
    </mirror>
```
![https://raw.githubusercontent.com/yangzhij/myself-images/master/maven3.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/maven3.png)
```
    <profile>
    <id>nexus</id>
    <repositories>
      <repository>
        <id>nexus</id>
        <name>Nexus</name>
        <url>http://172.20.100.202:8084/repository/maven-public/</url>
        <releases>
          <enabled>true</enabled>
        </releases>
        <snapshots>
          <enabled>true</enabled>
        </snapshots>
      </repository>
     </repositories>
   </profile>
```
![https://raw.githubusercontent.com/yangzhij/myself-images/master/maven4.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/maven4.png)
```
  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
```
![https://raw.githubusercontent.com/yangzhij/myself-images/master/maven5.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/maven5.png)

**#重启**  
```  
cd /home/Jenkinstomcat/bin
./startup.sh 
```              
**访问：**
http://172.20.100.176:83/jenkins/

现在！！  就可以登陆jenkins了，然后跟着提示一步步来就可以了。
**还有一种安装jenkins的方式是直接跑jenkins.war包**
1.安装java并配置
2.下载jenkins包并启动.....查看具体版本wget http://mirrors.jenkins-ci.org/war-stable/
mkdir /home/jenkins
cd /home/jenkins
wget http://mirrors.jenkins-ci.org/war-stable/2.121.3/jenkins.war
nohup java -jar ./jenkins.war --httpPort=81 >/dev/null 2>&1 &        #后台启动并指定端口
访问：http://47.90.122.91:81/
安装插件： 系统管理--安装插件--可选插件中搜索maven  	选择Maven Integration 和 Pipeline Maven Integration 安装即可。



























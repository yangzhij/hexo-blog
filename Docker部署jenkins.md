---
title: Docker部署jenkins
date: 2019-01-03 23:50:14
password: passw0rd
tags:
---
1. 创建工作目录
```
mkdir -p /mydocker/jenkins
```
2. 进入工作目录，把如下相关的软件包放到该目录下，此时我已经把jenkins.wa放在apache-tomcat里了,并且Tomcat的端口已改成83。建议以后做之前也这样先设置一下，此时端口改不改倒无所谓，反正最后也会再映射一个端口。
![https://raw.githubusercontent.com/yangzhij/myself-images/master/docker-jenkins1.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/docker-jenkins1.png)
3. 创建Dockerfile
```
[root@localhost jenkins]# more Dockerfile 
FROM  centos
MAINTAINER      remy<123456@gmail.com>
#把宿主机当前目录下的test.txt文件拷贝到容器/usr/local/路径下，重命名为docker_test.txt
#COPY test.txt /usr/local/docker_test.txt
#把当前目录下的java与Tomcat压缩包拷贝到到容器中并解压
ADD java.tar.gz /usr/local/
ADD apache-maven-3.5.4.zip /usr/local/
ADD tomcatjenkins.tar.gz /home
#安装vim编辑器
#RUN yum -y install gcc-c++
#RUN yum install -y pcre pcre-devel
#RUN yum install -y zlib zlib-devel
#RUN yum install -y openssl openssl-devel
RUN yum -y install fipscheck-lib
RUN yum install -y git
RUN curl -sL https://rpm.nodesource.com/setup_8.x | bash -
RUN yum -y install nodejs
#设置进入容器后默认的路径，即登录进入的落脚点
ENV mypath /home/
WORKDIR $mypath
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV MAVEN_HOME /usr/local/apache-maven-3.5.4
ENV PATH $PATH:$MAVEN_HOME/
ENV CATALINA_HOME /home/Jenkinstomcat
ENV CATALINA_BASE /home/Jenkinstomcat
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#容器运行时监听的端口
EXPOSE 83
#启动容器时运行tomcat
#ENTRYPOINT ["/usr/local/apache-tomcat-7.0.90/bin/startup.sh"]
CMD /home/Jenkinstomcat/bin/startup.sh && tail -F /home/Jenkinstomcat/bin/logs/catalina.out
[root@localhost jenkins]# 
```
需要手动进入解压一下zip的包，这个他娘的不能自行解压
4. 在当前目录下将dockerfile 构建成镜像
```
docker build -t my_jenkins .
```
5. 运行镜像
```
docker run -d -p 84:83 -v /mydocker/jenkins/test-jobs:/root/.jenkins/jobs --privileged=true --restart=always my_jenkins
```
6. 运行完后，通过docker ps查看，如果可以看到容器，就可以访问了   http://ip:84/jenkins
7. 访问jenkins, 会要求输入密码，这个直接进入容器，more 一下路径拷贝下就可以了。
8. 如果要迁移服务，就把相关服务放到宿主机的/mydocker/jenkins/test-jobs下面，这样容器中jenkins下面/root/.jenkins/jobs对应的就有了，最后在重启一下容器。
9. 进入和重启jenkins
```
docker exec -it 容器id /bin/bash            //进入        ctrl+p+q 退出
docker restart 容器id                   //重启
```




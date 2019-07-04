---
title: Docker基本命令
date: 2018-12-29 00:54:06
tags:
---
1.帮助命令：
``` 
docker version  //版本   docker info  //信息    docker --help   //帮助手册
```

2.镜像命令：一个镜像可以生成多个容器实例。
``` 
docker images   #列出当前主机可运行的镜像模板   -a 全部   -q 只显示当前镜像id  -qa 全部
镜像仓库源    镜像标签     镜像ID   镜像创建时间    镜像大小
[root@myself ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              1e1148e4cc2c        3 weeks ago         202MB

docker search  镜像名     如docker search tomcat    会在hub.docker.com查找    
docker search -s 30 tomcat   列出点赞数超过30的tomcat镜像

下载镜像：docker pull <镜像名:tag>    如docker pull tomcat  等价于 docker pull tomcat:latest   拉取的是最新的
指定版本: docker pull tomcat:8.2       拉取8.2版本

删除镜像：
docker rmi 镜像名        //这个是删除最新的；镜像名加上版本号就删除指定版本，直接用镜像id也可
删除单个： docker rmi -f 镜像id/镜像名
删除多个： docker rmi -f 名1 名2
删除全部： docker rmi -f $(docker images -qa)
```

3.容器命令
```
拉取镜像：docker pull <镜像名:tag>       eg. docker pull centos
查看镜像：docker images 

启动：  -i  交互式窗口，与t连用   -t 伪终端  --name 指定别名，不加该参数会随机生成一个名称
	a. //新建并启动交互式容器，进入容器
docker run -it --name 别名 镜像名/id    eg. docker run -it --anme mycentos1.1 centos
	b. //守护式进程启动（后台启动），不进容器
docker run -d --name 别名 镜像名/id     eg. docker run -d --name mytomcat tomcat

查看正在运行的容器：
docker ps
	
退出容器：
	a. //关闭并退出  exit
重新进入要启动：docker start 容器id或容器名
[root@myself ~]# docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
cbc0f86f8022        centos              "/bin/bash"         8 minutes ago       Exited (0) 40 seconds ago                       mycentos
[root@myself ~]# docker start cbc0f86f8022 
cbc0f86f8022
[root@myself ~]# docker attach cbc0f86f8022
[root@cbc0f86f8022 /]# 	
	b. //不关闭容器退出    ctrl+p+q
重新进入：docker attach 容器id

容器可以被启动、重启、停止、删除
docker start/restart/stop/kill 容器id或容器名     kill --快速强制停止
删除已停止的容器
docker rm  -f 容器id或容器名
docker rm -f $(docker ps -q)   //删除全部

执行命令
docker exec -it 容器id ls -l /tmp    在外面执行命令并返回结果
docker exec -it 容器id /bin/bash     进入容器再执行
ls -l /tmp

[root@myself ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cbc0f86f8022        centos              "/bin/bash"         12 minutes ago      Up 3 minutes                            mycentos
[root@myself ~]# docker exec -it cbc0f86f8022 ls -l /home/
total 0
drwxr-xr-x 2 root root 6 Dec 28 17:11 remy
[root@myself ~]# docker exec -it cbc0f86f8022 /bin/bash
[root@cbc0f86f8022 /]# ls -l /home/
total 0
drwxr-xr-x 2 root root 6 Dec 28 17:11 remy
[root@cbc0f86f8022 /]# 

查看容器日志：
docker log -t -f --tail  容器id

拷贝容器内的文件到主机
[root@myself ~]# docker cp cbc0f86f8022:/home/remy/20181229 /root/     容器id:容器内路径  容器外虚拟机路径

    
提交容器副本使之成为一个新的镜像：   docker commit  
docker commit -m="提交的描述信息" -a="作者" 容器id  要创建的目标镜像名:[标签名]
docker commit -a="remy" -m="im testing" 6c719cc2bb1c mytomcat:1.1
docker run -d mytomcat:1.1   //后台启动tomcat
步骤：
	1.先运行一个容器,如docker run -d -p 8188:8080 tomcat //指定外部映射端口    docker run -it -P tomcat   //随机分配外部端口  
	2.进入容器操作
	3.将更改过配置的容器生成一个新的镜像
	4.运行新的镜像   记得加标签

```


























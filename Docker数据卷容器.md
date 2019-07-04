---
title: Docker数据卷容器
date: 2018-12-29 06:21:51
tags:
---
**数据卷：**可以完成容器到主机，主机到容器的数据共享.主要用于数据持久化
docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据作为镜像的一部分保存下来，那么当容器删除后，数据就没了。所以为了能保存数据在docker中做数据持久化，我们使用数据卷。
**数据卷的添加：** 
1. 直接用命令添加
```
docker run -it -v /宿主机绝对路径目录:/容器内目录 (--privileged=true)  镜像名          //读写
docker run -it -v /宿主机绝对路径目录:/容器内目录:ro  镜像名                 //容器内只读

docker run -it -v /home/datahome:/datadocker centos
docker run -it -v /home/datahome:/datadocker:ro centos	
```

2. dockerfile添加
```
dockerfile是对镜像的一种源码描述。Dockerfile添加数据卷的简单操作：

1.根目录下新建mydocker文件夹并进入     mkdir /mydocker

2.编辑Dockerfile文件，使用VOLUME指令来给镜像添加一个或多个数据卷
[root@localhost mydocker]# more Dockefile 
#volume test
FROM centos
VOLUME ["/home/data1","/home/data2"]          //此指令是在容器内新建两个数据目录来和外界交互
CMD echo "finshed,-------------success!"
CMD /bin/bash

3.file构建---把dockerfile文件build成一个新镜像，build后生成一个新镜像remy/centos(镜像名随意)   . 表示在当前目录下面
docker build -f /mydocker/Dockefile -t remy/centos .

4.run容器     docker run -it remy/centos
5.docker inspect 容器id      //查看对应的宿主机的目录
```
docker inspect 容器id      //查看，此时容器内目录就可以和宿主机对应的目录进行数据同步了，可以新建文件测试
![https://raw.githubusercontent.com/yangzhij/myself-images/master/docker-shujujuan.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/docker-shujujuan.png)


**数据卷容器：通过挂载父容器来实现数据的传递依赖，相当于一条绳上的蚂蚱，容器之间、容器与宿主机之间数据同步共享**
```
以之前的数据卷为基础

docker run -it --name dc01 remy/centos        运行之前生成的一个镜像，该镜像本身自带数据卷         

docker run -it --name dc02 --volumes-from dc01 remy/centos         --volumes-from  继承的容器名   镜像名

docker run -it --name dc03 --volumes-from dc01 remy/centos

现在的情况是：
dc01容器上面本身有data1和data2目录，因为dc02和dc03都继承dc01，所以2和3上也有。
如果把dc01的容器干掉，dc02和dc03的目录依然存在，在2或3上的数据卷目录下新建文件，彼此都会有。
只要存在1个，那就会继承。
如果新加一个dc04继承dc02，同样dc04也有，在dc04上新建文件，2和3也会有。
```
---
title: Docker简介及安装
date: 2018-12-29 00:47:02
tags:
---

devops：开发自运维。  这应该是你的方向。
docker基于Go语言实现的云开源项目。可以让程序带环境安装：通过docker封装好原始环境（代码，配置，数据，系统.......）
**docker中文网站： https://docker-cn.com**
**docker出现：**
经常性的运维开发扯皮。部署开发提交的代码（jar包/war包），开发本地可以，运维部署出现问题：环境和配置。
之前，开发交代码---有了docker以后，开发可以交镜像（把环境打包成镜像，一次封装，处处运行）

**为什么docker比虚拟机快呢？**
虚拟机模拟的是整套操作系统（软件+硬件）。对于底层系统而言，虚拟机就是一个文件。不用了就删除。不影响别的。分钟级启动。
Linux容器模拟的不是一整套操作系统，只对进程进行隔离。容器没有内核，也不虚拟硬件，共用宿主机的，运行速度快（秒级启动）。可以理解为高度浓缩版的liunx系统。系统上只有一个docker引擎，没有硬件资源，不加载内核，宿主机的资源统一用（与宿主机共享OS）。  干掉了操作系统和硬件虚拟资源。



**安装前提：  centos6.5以上版本，64位版本**  
Client端(shell终端，也叫docker客户端)：  docker build;    docker pull;   docker run; 
Docker_Host:  装docker的主机
Registry: 注册仓库， 需要运行什么，直接从仓库拉取镜像，镜像即容器实例。仓库注册服务器上会存放多个仓库。 


docker: 容器运行的载体，是一个管理引擎。  只有通过镜像文件才能生成docker容器。
ocker原理： c/s结构。  dockers守护进程运行在主机上，通过socket连接从客户端访问。守护进程从客户端接收命令并管理运行在主机上的机器，容器是一个运行时的环境，即集装箱。
**docker三要素：**
镜像： 模板，容器实例.
容器： 独立运行的一个或一组应用。  容器是用镜像创建的运行实例。容器间相互隔离。
仓库： 存放镜像文件。
仓库分公开库和私有库。有名的公开库docker hub： 放容器的仓库，国内基本用不到，因为是国外的网站，太慢。国内用阿里云和网易云。


**安装：**
centos6.8安装
```
yum install -y epel-release    #依赖库
yum -y install docker-io
安装后的配置文件：  vi /etc/sysconfig/docker
other_args="--registry-mirror=自己账号的加速信息"   #指定从国内阿里云下载镜像
启动docker后台服务： service docker start
验证： docker version
启动测试：docker run hello-world
```
**run干了什么?** 
先会在本机找镜像，如果本地有，以镜像为模板生成容器实例运行，没有，就去仓库查找该镜像，能找到，先下载到本地，以镜像为模板生成容器实例运行，没有则返回错误信息--查不到该镜像。

centos7安装docker
```
yum -y update
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo  (国外用dockerhub)
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo (国内用阿里云)
yum list docker-ce --showduplicates | sort -r              #查看所有仓库中所有docker版本，并选择特定版本安装
yum -y install docker-ce-17.12.1.ce                        #安装所选版本
yum -y install docker-ce            #直接安装最新版本，可忽略上两行命令                        
systemctl start docker              //启动docker
```


卸载docker
```
systemctl stop docker
yum remove docker-ce -y
rm -rf /var/lib/docker
```

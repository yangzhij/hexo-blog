---
title: centos7安装docker
date: 2018-12-26 00:54:13
password: passw0rd
tags:
---

**Docker三要素：**
镜像 image
容器 
仓库

**centos7安装docker**
```
yum -y update
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum list docker-ce --showduplicates | sort -r              #查看所有仓库中所有docker版本，并选择特定版本安装
yum -y install docker-ce-17.12.1.ce                        #安装所选版本
```

**docker基本信息查看**
```
docker version          #版本号
docker info             # 查看系统(docker)层面信息
docker search 镜像名    #搜索镜像
docker pull 镜像名      #下载镜像
docker images           #列出本地有的镜像
docker images -a        #列出所有的images（包含历史）
```
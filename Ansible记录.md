---
title: Ansible记录
date: 2018-12-28 03:21:11
tags:
---
**Ansible简单说明:**
用于配置管理的自动化工具，基于python开发，分布式，使用ssh来管理客户端。可以实现多节点的软件部署，程序部署，配置管理等。ansible不需要启动服务，批量管理远程主机，执行结果会在本地返回。


*//ansible 使用依赖于 ssh*
```
//在管理端输入此命令，直接回车，生成密钥对，在/root/.ssh/下有两个文件。一个公钥，一个私钥。然后把公钥分发至被管理主机上。
ssh-keygen     //连续回车生成密钥
ssh-copy-id root@10.0.0.1            //推送公钥到远程主机
```
开始安装ansible
```
//管理端
yum install epel-release  -y
yum install ansible -y

//若客户端开启SElinux，需安装插件，一般是关闭的
yum install libselinux-python  -y	
yum -y install python-devel
```
主机管理端配置文件 vim /etc/ansible/hosts   //在此配置文件中如下定义主机组，文件末尾追加即可。如下默认端口是22，特殊端口需指定
```
[db-test]
10.0.0.1
10.0.0.2  ansible_ssh_port=9999
[mq-test]
10.0.0.32          
10.0.0.33
10.0.0.3[2:3]   //也可这样写，和上面一样
[web]
10.11.0.25
10.11.0.26
[web:var]        //统一对web组设置变量
ansible_ssh_port=22
ansible_ssh_user=root
ansible_ssh_pass=123456
```
//command模块功能：在指定节点上运行linux命令
//可指定当个远程IP或者整个主机组
```
[root@localhost ~]# ansible 10.0.0.1 -m command -a 'bash /root/a.sh'
[root@localhost ~]# ansible db-test -m command -a 'bash /root/a.sh'
```
//creates：一个文件名，当该文件存在，则该命令不执行。如运行脚本，当远程主机上/home/ip存在，则不执行该脚本
```
[root@localhost ~]# ansible db-test -m command -a 'bash /root/a.sh creates=/home/ip'       
```
//removes：指定一个文件名，当远程主机上该文件不存在，则该命令不执行,存在则执行		  
```
[root@localhost ~]# ansible db-test -m command -a 'bash /root/a.sh removes=/home/ip'
```

//copy模块：把server端某一文件拷贝到指定节点上。若指定节点上有该文件，则覆盖，文件属性不变。
//src原文件路径
//dest目标文件路径， 可在路径中指定文件名，如果不指定则默认使用原文件名
//owner 指定属主
//group 指定属组
//backup=yes    在覆盖之前将远程主机的原文件备份，当文件内容不变时，使用backup，则不会进行备份
```
ansible 172.20.100.162 -m copy "src=/home/wwwroot/hello.jar dest=/home/wwwroot/hello.jar owner=root group=root mode=0644 backup=yes"  
```
//fetch模块功能：将远程主机中的文件拷贝到本机中
```
ansible 172.20.100.162 -m fetch -a 'src=/root/www.html dest=/home/remy/'
```

一些相关的基础用法
```
ansible db-test -m command -a "ls -l /server/scripts"            //查看远程主机文件

ansible db-test -m command -a "rpm -qa iotop"              //查看有没有某个安装包

ansible db-test -m shell -a "/server/scripts/a.sh"           //批量执行分发在被管理机上的脚本
ansible db-test -m command -a 'bash /root/a.sh'

ansible liantong –m copy -a "src=/server/scripts/yum.sh dest=/server/scripts/ mode=0755"     //复制文件到远程主机
```


<font color="#FF0000">**！！！重点来了---playbook**</font> 

**playbook基础组件**
•	Hosts：运行执行任务（task）的目标主机，可以是单个ip，也可是定义的主机组
•	remote_user：在远程主机上执行任务的用户
•	tasks：任务列表
•	handlers：任务，与tasks不同的是只有在接受到通知时才会被触发

playbook由YMAL语言编写，整个playbook是以task为中心，表明要执行的任务。hosts和remote_user表明在哪些远程主机以何种身份执行.
Name: 每一个task都有一个名称，用于标记此任务。

文件名以yaml结尾，如a.yaml   执行命令为ansible-playbook a.yml
```
[root@localhost yamls]# more a.yaml 
- hosts: 172.20.100.68
  remote_user: root
  
  tasks:
    - name: copy jar to remote
      copy: src=/home/pay-1.0.0.jar  dest=/home/pay-1.0.0.jar  owner=root  group=root  mode=0644

    - name: copy shell to remote
      copy: src=/home/shell/pay-service.sh  dest=/home/wwwroot/pay-service.sh  owner=root group=root mode=0755
    
    - name: execute shell 
      shell: /bin/bash /home/wwwroot/pay-service.sh

[root@localhost yamls]# 
```
![https://raw.githubusercontent.com/yangzhij/myself-images/master/yaml.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/yaml.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/jenk-ssh.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/jenk-ssh.png)











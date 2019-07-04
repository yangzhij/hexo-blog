---
title: centos7 zabbix安装
date: 2018-12-19 16:29:11
password: passw0rd
tags:
---

firewall:
```
查看防火墙状态：firewall-cmd --state
关闭：systemctl stop firewalld.service
禁止开机自启：systemctl disable firewalld.service
```
selinux:
```
临时关闭：setenforce 0
永久关闭：vi /etc/selinux/config，将SELINUX=enforcing改为SELINUX=disabled;重启生效
```
Zabbix安装
1.下载yum源: #查看具体版本http://repo.zabbix.com/zabbix/
```
//二选一
rpm -ivh http://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
rpm -ivh http://repo.zabbix.com/zabbix/4.1/rhel/7/x86_64/zabbix-release-4.1-1.el7.noarch.rpm
```
2.安装：
```
yum install -y zabbix-server-mysql zabbix-web-mysql zabbix-agent mariadb mariadb-server
```
3.启动mysql并设置开机启动
```
systemctl start mariadb
systemctl enable mariadb
```
4.进入mysql创建实例并授权
```
MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;   #创建数据库实例
MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@'localhost' identified by 'zabbix';  #授权访问数据库实例zabbix
MariaDB [(none)]> flush privileges;    #刷新保存
```
5.导入出始表数据
```
cd /usr/share/doc/zabbix-server-mysql-4.0.1/        #doc后面的目录取决于安装的哪个版本
zcat create.sql.gz |mysql -uroot zabbix             #导入
```
6.修改zabbix的配置
vi /etc/zabbix/zabbix_server.conf
```
DBHost=localhost         # 数据库主机名（只改这个就行，后面3个都是默认的）
DBName=zabbix            # 数据库实例
DBUser=zabbix            # 用户名
DBPassword=zabbix        # 密码
```
启动：
```
systemctl start zabbix-server           #启动zabbix-server服务
systemctl enable zabbix-server       #设置zabbix-server服务开机自启动
```
7.安装apache,并修改Apache的配置文件
```
//安装
yum -y install httpd
//修改配置
vi /etc/httpd/conf.d/zabbix.conf
修改为：php_value date.timezone Asia/Shanghai
```
启动httpd服务 ，设置开机启动httpd服务
```
systemctl start httpd
systemctl enable httpd
```
启动zabbix-agent并设置开机自启动
```
systemctl start zabbix-agent
systemctl enable zabbix-agent
```

在浏览器输入地址http://172.20.100.115/zabbix/setup.php

管理员用户名Admin(区分大小写)，默认密码zabbix
默认页面为英文，点击右上角头像，将语言改为中文，主题Theme改为黑色，修改完成以后退出登录重进即可。


**客户端安装：**  <font color="#FF0000">*注意master-slave版本要一致* </font>
```
rpm -ivh http://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
yum install zabbix-agent -y
sed -i 's/Server=127.0.0.1/Server=172.20.100.115/' /etc/zabbix/zabbix_agentd.conf   #服务端ip
sed -i 's/ServerActive=127.0.0.1/ServerActive=172.20.100.115/' /etc/zabbix/zabbix_agentd.conf     #服务端IP
sed -i 's/Hostname=Zabbix server/Hostname=172.20.100.124/' /etc/zabbix/zabbix_agentd.conf         #本机IP
systemctl enable zabbix-agent.service
systemctl start zabbix-agent.service
```
然后就可以在服务器添加主机IP了，添加群组，添加模板等等.....



**Zabbix图像中文乱码处理**: 将windows中的字体，替换zabbix php 中的字体即可。
```
打开 windows 控制面板——》字体——》如选择 “黑体”——》上传到linux中如下目录
[root@elk-node1 dejavu]# find / -name 'DejaVuSans.ttf'
/usr/share/fonts/dejavu/DejaVuSans.ttf
[root@elk-node1 dejavu]# cd /usr/share/fonts/dejavu/
将simhei.ttf上传到该目录下
[root@elk-node1 dejavu]# mv DejaVuSans.ttf DejaVuSans.ttf.bak
[root@elk-node1 dejavu]# mv simhei.ttf DejaVuSans.ttf
刷新下页面就可以了
```










---
title: ftp记录
date: 2018-12-13 08:56:01
tags:
---

FTP：用于文件上传下载。FTP采用客户/服务器模式，客户机与服务器之间利用TCP建立连接，客户可以从服务器上下载文件，也可以把本地文件上传至服务器。
```
vi /etc/selinux/config
SELINUX=enforcing 设置成SELINUX=disabled

systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
firewall-cmd --state #查看默认防火墙状态
```

安装
```
yum install vsftpd
mkdir -p /var/data/flower/{public_html,logs,backup}   #创建工作目录
useradd flower -d /var/data/flower         #创建指定用户的工作目录，即用户登陆后直接进入该目录
echo "123456" | passwd flower --stdin       #设置用户密码为123456
```

修改基本配置配置 vsftpd的配置文件/etc/vsftpd/vsftpd.conf 
```
sed -i 's/anonymous_enable=YES/anonymous_enable=NO/' /etc/vsftpd/vsftpd.conf                   
sed -i 's/#chroot_local_user=YES/chroot_local_user=YES/' /etc/vsftpd/vsftpd.conf
sed -i 's/#chroot_list_enable=YES/chroot_list_enable=YES/' /etc/vsftpd/vsftpd.conf       //开启chroot_list
anon_mkdir_write_enable=YES     #可以新建文件夹
```

添加用户
```
在/etc/vsftpd/chroot_list 中写入 flower 用户
```

然后给目录权限，可以通过属组来限制用户上传
```
chown -R flower.flower /var/data/flower/
chmod -R 777 /var/data/flower/
```

重启服务
```
systemctl restart vsftpd
```

线上ftp配置：
```
anonymous_enable=NO     #禁止匿名用户访问 
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES          #dirmessage_enable=YES
xferlog_enable=YES            #启用记录上传/下载活动日志功能。
connect_from_port_20=YES          启用FTP数据端口的连接请求
xferlog_std_format=YES             #用标准的ftpdxferlog日志格式
chroot_local_user=YES           #将所有用户限制在工作目录
chroot_list_enable=YES             ##启动限制用户的名单
chroot_list_file=/etc/vsftpd/chroot_list
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd
userlist_enable=YES   #此项配置/etc/vsftpd.user_list中指定的用户也不能访问服务器
tcp_wrappers=YES
allow_writeable_chroot=YES
```



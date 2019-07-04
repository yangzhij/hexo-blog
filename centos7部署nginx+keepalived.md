---
title: centos7部署nginx+keepalived
date: 2018-12-27 02:27:11
password: passw0rd
tags:
---

**<font color="red">nginx代理后端服务，Keepalived用来检测服务器的状态</font> **

双机热备高可用nginx+keepalived，方式有两种：
1.主备：一个vip，同一时间只有一台主机对外提供服务，另外一台做备用机。
2.主主：二个VIP或多个，两台主机都对外提供服务，可以是同一个服务，也可以是不同的服务，两个VIP互为主备模式。


**部署环境：**

10.0.0.5    nginx+keepalievd
10.0.0.6	nginx+keepalievd

vip1: 10.0.0.105
vip2: 10.0.0.106

<font color="green">本次采用主主来配置多个VIP。在10.0.0.5和10.0.0.6上安装以下过程安装nginx+keepalievd，注意10.0.0.6的keepalived.conf文件配置。操作记录如下</font> 
**安装编译nginx**
```
//依赖
yum  -y groupinstall "Development Tools" "Server Platform Deveopment"
yum  -y install openssl-devel pcre-devel
//添加nginx用户
useradd -M -s /sbin/nologin nginx
//下载解压
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.12.0.tar.gz
tar -zxvf nginx-1.12.0.tar.gz
//编译安装
cd nginx-1.12.0
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre
make&& make install
```

**编辑nginx配置文件**
cd /usr/local/nginx/conf/
mv nginx.conf nginx.conf.bak
vi nginx.conf      //输入以下内容，已经优化好的配置，直接用即可
```
user nginx;

worker_processes auto;

error_log  /var/log/nginx/error.log warn;

pid        /var/run/nginx.pid;

events 
    {
        use epoll;
        worker_connections 51200;
        multi_accept on;
    }

http
    {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;

        server_names_hash_bucket_size 128;
        client_header_buffer_size 32k;
        large_client_header_buffers 4 32k;
        client_max_body_size 256M;

        log_format  main  '$remote_addr - [$time_local] "$request" '
             '$status $body_bytes_sent $request_time "$http_referer" '
             '"$http_user_agent" - $http_x_forwarded_for';

        
        sendfile        on;
        tcp_nopush     on;

        keepalive_timeout 1800s 1800s;
        keepalive_requests 10240;

        gzip on;
        gzip_min_length  1k;
        gzip_buffers     4 16k;
        gzip_http_version 1.1;
        gzip_comp_level 6;
        gzip_types     text/plain application/javascript application/x-javascript text/javascript text/css application/xml application/xml+rss;
        gzip_vary on;
        gzip_proxied   expired no-cache no-store private auth;
        gzip_disable   "MSIE [1-6]\.";

        access_log off;
        access_log  /var/log/nginx/access.log;
        
        include /usr/local/nginx/conf.d/*.conf;    
        include /usr/local/nginx/conf.d/game/*.conf;
    
    }

```
include是包含的配置文件,如下，具体的配置可以在conf.d下配置,如果要在conf.d目录下新建目录，需要在上述配置文件中追加一行
```
[root@localhost conf.d]# ll
total 344
-rw-r--r-- 1 root root  882 Dec 14 20:41 xiaoa.dangxiang.com.conf
-rw-r--r-- 1 root root 1245 Dec 16 16:28 agfront.dangxiang.com.conf
-rw-r--r-- 1 root root  731 Dec 17 11:04 ag.test100.com.conf
-rw-r--r-- 1 root root  731 Dec 19 11:17 ag.test101.com.conf
-rw-r--r-- 1 root root  620 Nov 16 20:11 app.daxiang.com.conf
-rw-r--r-- 1 root root  824 Dec 14 20:46 haha.daxiang.com.conf
```
配置内容举例
```
[root@localhost conf.d]# more xiaoa.dangxiang.com.conf
upstream xiaoa { 
      server 172.20.100.159:6003 weight=2 max_fails=3 fail_timeout=10s; 
      server 172.20.100.160:6003 weight=2 max_fails=3 fail_timeout=10s;
      keepalive 1024;
}


server {
    listen 80;
    server_name xiaoa.dangxiang.com;

    location / {
        #proxy_pass http://172.20.100.214:5200;
        proxy_pass http://xiaoa;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-real-ip $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;

        proxy_connect_timeout 1800s;
        proxy_read_timeout 1800s;
        proxy_send_timeout 1800s;
    }

    access_log /var/log/nginx/xiaoa.dangxiang.com.log;
}
```
**启动nginx**
```
[root@localhost conf.d]#  /usr/local/nginx/sbin/nginx
```
**添加环境变量**
```
在/etc/profile 中加入：
export NGINX_HOME=/usr/local/nginx
export PATH=$PATH:$NGINX_HOME/sbin
保存，执行 source /etc/profile ，使配置文件生效。
[root@elk-node1 home]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@elk-node1 home]# nginx -s reload
[root@elk-node1 home]# 
```
**安装keepalived并修改配置文件**
安装： yum install -y keepalived
配置文件：vi /etc/keepalived/keepalived.conf 
```
[root@localhost keepalived]# more keepalived.conf 
vrrp_script chk_port {
    script "/root/scripts/check_nginx.sh"
    interval 2
}
vrrp_instance VI_1 {
    interface eth0
    state MASTER
    nopreempt
    priority 100
    virtual_router_id 20
    authentication {
        auth_type PASS
        auth_pass password
    }
    virtual_ipaddress {
        10.0.0.105
    }
}
vrrp_instance VI_2{  
   state BACKUP
   interface ens33
   virtual_router_id 21
   priority 80
   advert_int 1  
   authentication {  
       auth_type PASS
       auth_pass 1111  
   }
   virtual_ipaddress {
       10.0.0.106
}
track_script {                     
   chk_port                 
}
}

```

**配置nginx检测脚本** 
```
vi /root/scripts/check_nginx.sh
#!/bin/bash
NGINX=`ps-C nginx --no-heading| wc -l`
if[ "${NGINX}" = "0" ]; then
        service stop keepalived
fi

```

1.复制10.0.0.5上的keepalived.conf 到10.0.0.6上并进行修改;
2.在10.0.0.6上修改nginx配置文件和编写nginx状态检测脚本同上;
```
[root@localhost keepalived]# more keepalived.conf 
vrrp_script chk_port {
    script "/root/scripts/check_nginx.sh"
    interval 2
}
vrrp_instance VI_1 {
    interface eth0
    state BACKUP
    nopreempt
    priority 80
    virtual_router_id 20
    authentication {
        auth_type PASS
        auth_pass password
    }
    virtual_ipaddress {
        10.0.0.106
    }
}
vrrp_instance VI_2{  
   state MASTER
   interface ens33
   virtual_router_id 21
   priority 100
   advert_int 1  
   authentication {  
       auth_type PASS
       auth_pass 1111  
   }
   virtual_ipaddress {
       10.0.0.106
}
track_script {                     
   chk_port                 
}
}

```

**启动keepalived**
```
systemctl start keepalived
```
至此，安装完毕！

说明：
	1. 105上启动keepalived，通过ip addr查看会有两个vip，当主机10.0.0.106启动keepalived后，vip2会被抢占过去。
	2. keepalived配置文件会自动执行监控nginx进程的脚本看是否要停止本机的keepalived服务，从而转移到正常的一台。






















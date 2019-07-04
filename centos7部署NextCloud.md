---
title: centos7部署NextCloud
date: 2019-07-04 14:15:30
tags:
---
//检查下列服务若自带则卸载（rpm -e ... --nodeps）
```
rpm -qa|grep php
rpm -qa|grep php-common
rpm -qa|grep nginx
```

//安装相关组件；并检查php-fpm是否已正常安装
```
yum -y install epel-release
yum -y install nginx
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum -y install php70w-fpm php70w-cli php70w-gd php70w-mcrypt php70w-mysql php70w-pear php70w-xml php70w-mbstring php70w-pdo php70w-json php70w-pecl-apcu php70w-pecl-apcu-devel
php -v
```

配置php-fpm
```
sed -i 's/user = apache/user = nginx/' /etc/php-fpm.d/www.conf
sed -i 's/group = apache/group = nginx/' /etc/php-fpm.d/www.conf
sed -i 's/;env/env/g' /etc/php-fpm.d/www.conf
```

在/var/lib目录下为session路径创建一个新的文件夹，并将用户名和组设为nginx
```
mkdir -p /var/lib/php/session
chown nginx:nginx -R /var/lib/php/session/
ll -d /var/lib/php/session/
```

启动Nginx和php-fpm服务，并添加开机启动
```
systemctl start php-fpm
systemctl start nginx
systemctl enable php-fpm
systemctl enable nginx
```

安装并配置MariaDB,启动并添加开机启动
```
yum -y install mariadb mariadb-server
systemctl start mariadb
systemctl enable mariadb
```

//接下来设置MariaDB的root密码
```
[root@nextcloud-server ~]# mysql_secure_installation        //按照提示设置密码，首先会询问当前密码，密码默认为空，直接回车即可
Enter current password for root (enter for none):          //直接回车
Set root password? [Y/n] Y
New password:                                              //输入新密码
Re-enter new password:                                     //再次输入新密码
     
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

//进入mysql   并为Nextcloud创建相应的用户和数据库。例如数据库为nextcloud_db，用户为nextclouduser，密码为cloudpasswd
```
[root@localhost ~]# mysql -p

MariaDB [(none)]> create database nextcloud_db;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create user nextclouduser@localhost identified by 'cloudpasswd';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all privileges on nextcloud_db.* to nextclouduser@localhost identified by 'cloudpasswd';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
```

//为Nextcloud生成自签名SSL证书
为SSL证书创建一个新的文件夹，然后将证书文件的权限设置为660 
```
[root@localhost ~]# mkdir /etc/nginx/cert/ && cd /etc/nginx/cert/
[root@localhost ~]# openssl req -new -x509 -days 365 -nodes -out /etc/nginx/cert/nextcloud.crt -keyout /etc/nginx/cert/nextcloud.key

Country Name (2 letter code) [XX]:cn                                           //国家
State or Province Name (full name) []:beijing                                  //省份
Locality Name (eg, city) [Default City]:beijing                                //地区名字
Organization Name (eg, company) [Default Company Ltd]:weiruan                       //公司名
Organizational Unit Name (eg, section) []:IT                           //部门
Common Name (eg, your name or your server's hostname) []:weiruan                 //CA主机名
Email Address []:weiruan@gmail.com           

chmod 700 /etc/nginx/cert
chmod 600 /etc/nginx/cert/*
```

//下载并安装Nextcloud
```
yum -y install wget unzip
cd /usr/local/src/
wget https://download.nextcloud.com/server/releases/nextcloud-12.0.4.zip
unzip nextcloud-12.0.4.zip
mv nextcloud /usr/share/nginx/html/
cd /usr/share/nginx/html/
mkdir -p nextcloud/data/
chown nginx:nginx -R nextcloud/
```

//设置Nginx虚拟主机
进入Nginx的虚拟主机配置文件所在目录并创建一个新的虚拟主机配置（记得修改两个server_name为自己的域名）
```
[root@localhost ~]# cd /etc/nginx/conf.d/ 
[root@localhost ~]# vim nextcloud.conf      //复制下面内容

upstream php-handler {
    server 127.0.0.1:9000;
}
     
server {
    listen 80;
    server_name nextcloud.remy.com;
    return 301 https://$server_name$request_uri;
}
     
server {
    listen 443 ssl;
    server_name nextcloud.remy.com;
     
    ssl_certificate /etc/nginx/cert/nextcloud.crt;
    ssl_certificate_key /etc/nginx/cert/nextcloud.key;
     
    add_header Strict-Transport-Security "max-age=15768000;
    includeSubDomains; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
     
    # Path to the root of your installation
    root /usr/share/nginx/html/nextcloud/;
     
    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
     
    location = /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
     
    # set max upload size
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;
     
    # Disable gzip to avoid the removal of the ETag header
    gzip off;

     
    error_page 403 /core/templates/403.php;
    error_page 404 /core/templates/404.php;
     
    location / {
        rewrite ^ /index.php$uri;
    }
     
    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
        deny all;
    }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }
     
    location ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+|core/templates/40[34])\.php(?:$|/) {
        include fastcgi_params;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        #Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }
     
    location ~ ^/(?:updater|ocs-provider)(?:$|/) {
        try_files $uri/ =404;
        index index.php;
    }
     
    location ~* \.(?:css|js)$ {
        try_files $uri /index.php$uri$is_args$args;
        add_header Cache-Control "public, max-age=7200";
        # Add headers to serve security related headers (It is intended to
        # have those duplicated to the ones above)
        # Before enabling Strict-Transport-Security headers please read into
        # this topic first.
        add_header Strict-Transport-Security "max-age=15768000;includeSubDomains; preload;";
        add_header X-Content-Type-Options nosniff;
        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        # Optional: Don't log access to assets
        access_log off;
    }
     
    location ~* \.(?:svg|gif|png|html|ttf|woff|ico|jpg|jpeg)$ {
        try_files $uri /index.php$uri$is_args$args;
        # Optional: Don't log access to other assets
        access_log off;
    }
}
```


//测试配置文件是否有误，确保没有问题后重启Nginx服务。
```
[root@localhost conf.d]# nginx -t
[root@localhost conf.d]# nginx -s reload
[root@localhost conf.d]# systemctl restart nginx
```

//nextcloud上传文件大小的自身限制为512M，修改php.ini上传文件大小限制
```
sed -i 's/max_execution_time = 30/max_execution_time = 0/' /etc/php.ini
sed -i 's/post_max_size = 8M/post_max_size = 10800M/' /etc/php.ini
sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 10240M/' /etc/php.ini
```

//解析上面nginx中配置的域名nextcloud.remy.com，浏览器访问nextcloud.remy.com进行Nextcloud界面安装

参考https://www.cnblogs.com/kevingrace/p/8343060.html













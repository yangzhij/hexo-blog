---
title: win10安装mysql
date: 2019-01-18 20:19:57
tags:
---
win10安装mysql

//下载
https://dev.mysql.com/downloads/mysql/ 

//解压:在你想安装的目录下解压，本次安装在E:\Mysql\ 下

//配置环境变量
系统变量
```
path
E:\Mysql\mysql-8.0.13-winx64\bin
```
//在E:\Mysql\mysql-8.0.13-winx64新建my-default.ini，内容如下
```
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=E:\Mysql\mysql-8.0.13-winx64
# 设置mysql数据库的数据的存放目录
datadir=E:\Mysql\mysql-8.0.13-winx64\data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

// cmd 以管理员身份运行，开始装及初始化ySQL服务
```
C:\Users\Admin>E:

E:\>cd E:\Mysql\mysql-8.0.13-winx64\bin

//安装
mysqld --install
//初始化
mysqld --initialize --console
```
初始化完后会生成一个原始密码。其中root@localhost:后面的"Ng*jbnKGd2_!"就是初始密码（不含首位空格）。在没有更改密码前，需要记住这个密码，后续登录需要用到。
如果没保存住，也没关系。删掉初始化的 data 目录，再执行一遍初始化命令，又会重新生成的。

//启动
```
net start mysql
```
//进入
```
mysql -u root -p     然后输入之前保存的密码
```
//修改用户密码
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';

CREATE USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY 'test123';            //创建用户

GRANT ALL PRIVILEGES ON *.* TO 'test'@'%'；  // #授权所有权限 

#授权基本的查询修改权限，按需求设置
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON *.* TO 'test'@'%';   

flush privileges;    //刷新保存
```
然后  工具就可以连接了


只要在CMD里输入一条命令就可以将服务删除：
sc delete mysql //这里的mysql是你要删除的服务名
这样一来服务就被删除了。



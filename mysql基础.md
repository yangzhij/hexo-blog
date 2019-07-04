---
title: mysql基础
date: 2018-12-28 01:12:42
tags:
---

MySQL中使用InnoDB存储引擎的时候一张表对应着两个物理文件，分别为frm（存储表结构）和ibd（存储数据）。
外键的主要作用是：保持数据的一致性、完整性。

//Mysql数据备份及恢复
```
备份：
mysqldump –uroot –p123456 --all-databases > all.sql             //备份全库
mysqldump –uroot –p123456 danku > danku.sql            //备份单库
Mysqldump –uroot –p123456 –d danku > biaojg.sql               //备份表结构
导出单个数据表结构：mysqldump -uroot -pxxx -d 库名 表名 > db.sql
导出某个数据库结构：mysqldump -uroot -p123456  -d 库名 > dump.sql
去掉参数-d 就是导出结构和数据

恢复：
mysql –h 172.10.10.1 –uroot –p123456 </home/all.sql
或者进入数据库：
mysql –uroot –p
#导入单库需要先指定库名   use abc_db;
mysql>source /home/all.sql    （注意不能中断，中断重新来过）  

Mysql数据恢复到新库
如果是备份单个库或某几个库，需要先去B库上创建与A库同样的库名（一般用客户端工具）。然后进入该库，执行source语句。或者mysql -u用户名 -p密码 danku < danku.sql
```

//Mysql创建用户并授权
```
create user 'myuser'@'%' identified by 'mypassword';    创建mysql用户并指定密码
grant select on *.* to 用户名@'%' identified by '密码';    赋予用户查看所有库的权限
grant select(id, name, tel) on 库名.表名 to 用户名@'%' identified by '密码';    赋予用户查看某张表其中几个字段的权限（select,insert,update,delete,create,drop ）
grant all privileges on 库名.* to '用户名'@'%' identified by '密码';  赋予用户对某个库的权限
revoke all on *.* from 用户名@'%';    回收权限
delete from mysql.user where user='用户名' and host='主机名或者%';   删除远程用户
flush privileges;   刷新更改
grant select on testdb.* to dba@localhost with grant option;（如果想让授权的用户再授权给别人，添加with grant option，一般不会用到）
grant select, insert, update, delete on 库名.* to 用户名@'%' ;   赋予远程用户对某个库下所有表的增删改查权限

host为可以登录的主机地址，如果任何主机都可以，设置为%
privileges - 用户的操作权限,如SELECT , INSERT , UPDATE 等(详细列表见该文最后面).如果要授予所的权限则使用ALL;
create database if not exists 库名 default charset utf8 collate utf8_general_ci;   创建数据库并指定字符集

//允许root用户远程登录
MariaDB [(none)]>grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
MariaDB [(none)]> flush privileges;

//创建一个用户db3只允许查询DB2和DB3库
MariaDB [(none)]> grant select on DB2.* to db3@'%' identified by '12345678';
MariaDB [(none)]> grant select on DB3.* to db3@'%' identified by '12345678';
MariaDB [(none)]> flush privileges;

//如果要配置任何主机通过myuser用户连接时不需要密码的话，可以使用以下命令
MariaDB [(none)]> create user 'myuser';   #这一步可不用
MariaDB [(none)]> grant all on *.* to myuser@'%' identified by '';
MariaDB [(none)]> flush privileges;
[root@localhost ~]# mysql –umyuser    直接进入

//查看用户：
MariaDB [(none)]> select * from mysql.user \G;

//删除用户：
MariaDB [(none)]> drop user 'gsdfghdfsgkhjk'@'%';
MariaDB [(none)]> select * from mysql.user where user='gsdfghdfsgkhjk' \G;

```

//备份脚本
备份过程中压缩会降低备份的速度。
mysqldump备份大数据库主要依赖于硬件，包括可用的内存和硬盘驱动器速度。一般备份在 5GB 和 20GB 之间适当的数据库大小。
```
#!/bin/bash
time=`date +%Y%m%d`
#年月日分时秒
ttime=`date +"%Y%m%d%H%M%S"`
DB_LIST=`echo "show databases;"|mysql  -uroot -p123456 |sed '1,4d'`
nodeldb="sys"

for dbname in $DB_LIST
do
	if [ $dbname != $nodeldb ];then
		/usr/bin/mysqldump -uroot -p123456  $dbname > /home/shenzhenbak/$dbname$time.sql 2>&1 &
	fi
done

--------------------------------------------------------------------------------------------------------------

#!/bin/bash
time=`date +%Y%m%d`
usr="root"
passwd="123456"
dir='/home/shenzhenbak'
mysqldump -u$usr -p$passwd shenzhendb > $dir/shenzhen$time.sql 2>&1 &


#截断日志
#mysql -u$usr -p$passwd -e 'flush logs'


#只保留最近一次的备份
#cd $$dir/shenzhen/
# find . -mtime +7 -exec rm -rf {} \;
```














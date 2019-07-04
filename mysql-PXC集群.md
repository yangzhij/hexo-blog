---
title: mysql_PXC集群
date: 2019-05-15 11:09:42
tags:
---
mysql PXC集群
-----至少3个节点，便于故障恢复
-----同步复制，集群中每个节点都包含完整数据
-----可以从集群中分离某节点单独使用。对于集群中新节点的加入，维护起来很简单。
-----PXC可以实现多个节点间的数据同步复制以及读写，并且保证了数据库的服务高可用及数据强一致性。
-----写性能依赖最慢的那个机器，因为PXC集群采用的是强一致性原则，一个更改操作在所有节点都成功才算执行成功。

//部署如下
从第1步到第11步，每个节点都要做
```
先修改3台机器的hostname
[root@localhost ~]# hostname set-hostname pxc_node1
修改hosts文件，三台机器都修改如下所示： 
172.20.100.185 pxc_node1
172.20.100.186 pxc_node2
172.20.100.187 pxc_node3

1.vi /etc/selinux/config
SELINUX=disabled   #修改该项为disabled

2.setenforce 0

3.systemctl status firewalld         //查看防火墙状态
如果是开启的，则开放端口3306 、4444、4567、4568
firewall-cmd --add-port=3306/tcp --permanent     #开放了3306端口
重新加载防火墙规则
firewall-cmd --reload

4.安装Persona仓库
yum -y install http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm

yum -y update percona-release

5.安装PXC（保证服务器没有装MySQL） 
yum -y install Percona-XtraDB-Cluster-57

6.开启PXC服务
systemctl start mysql

7.查看安装数据库的临时密码并记住
grep 'temporary password' /var/log/mysqld.log

8.登录MySQL数据库
mysql -u root -p

9.登录成功后修改密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '你的密码';
mysql> flush privileges; 
10.停止MySQL服务
systemctl stop mysql   （某些版本使用mysqld）


11.开始配置节点
--修改配置文件
vi  /etc/percona-xtradb-cluster.conf.d/wsrep.cnf
#集群中节点的IP地址（本机填最后）
wsrep_cluster_address=gcomm://ip地址,IP地址,IP地址（用,号隔开）

binlog_format=ROW

default_storage_engine=InnoDB

wsrep_slave_threads= 8

wsrep_log_conflicts

innodb_autoinc_lock_mode=2

wsrep_node_address=当前节点IP地址

#集群名称
wsrep_cluster_name=pxc-cluster         

#当前节点名称
wsrep_node_name=pxc-cluster-node-1

#不使用实验功能
pxc_strict_mode=ENFORCING

#状态快照传输（sst）方法，官方建议
wsrep_sst_method=xtrabackup-v2

#用户凭证（mysql的用户名和密码）
wsrep_sst_auth="用户名:密码"

剩下的节点修改当前节点名、当前节点IP、集群中的节点IP，其他相同
```

---------------------------------------------------------------------以上所有步骤   每个节点都要配置一次
//下面开始初始化集群节点
```
12.其中一个节点使用 systemctl start mysql@bootstrap.service 启动
登录mysql  

   mysql -u root -p

开启 wsrep_causal_reads

mysql> set wsrep_causal_reads =1;


13.创建配置文件中对应的用户   所有节点的IP都要创建

    
创建用户：   CREATE USER '用户名'@'localhost' IDENTIFIED BY '密码';  
刷新权限：   GRANT all privileges ON *.* TO '用户名'@'localhost' ;
                      FLUSH PRIVILEGES;

创建用户：   CREATE USER '用户名'@'当前需要访问数据库的IP地址' IDENTIFIED BY '密码';  
刷新权限：   GRANT all privileges ON *.* TO '用户名'@'当前需要访问数据库的IP地址' ;
                      FLUSH PRIVILEGES;

14.其他节点使用   systemctl start mysql  启动 ，登录mysql，配置wsrep_causal_reds，set wsrep_causal_reads =1;

15.其他节点启动成功后在引导节点（使用 systemctl start mysql@bootstrap.service 命令启动的节点）

验证集群：show status like 'wsrep%';  

------------到这里真的就搭建完了   可以试着在其中一台节点创建个数据库，这样别的节点就都有了

注意：服务的启动和停止要对应

        systemctl stop mysql   ------>  启动时用systemctl start mysql
或者 
        systemctl stop mysql@bootstrap.service   ----->  启用是用 systemctl start mysql@bootstrap.service 
```
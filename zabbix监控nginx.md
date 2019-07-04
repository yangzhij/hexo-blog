---
title: zabbix监控nginx
date: 2019-02-13 15:23:16
password: passw0rd
tags:
---
**zabbix监控nginx的性能状态，主要包括：**
nginx-accepts:  nginx所接受的客户端连接数
nginx-active：  当前活跃的客户端连接数
nginx-handled:  已成功的客户端连接数
nginx-requests：客户端请求数
nginx-ping
nginx-reading
nginx-waiting
nginx-writing
**---------------操作如下：前提：nginx运行正常**
cd /etc/nginx/conf.d/
新建status.conf 
```
[root@localhost zabbix]# more /etc/nginx/conf.d/status.conf 
server {
    listen 10080;
    access_log off;
    server_name localhost;
    root /var/www/html;

    location /nginx_status {
       stub_status on;
       access_log off;
       allow 172.20.100.0/24;
       deny all;

    }

}

```
cd /etc/zabbix/
more nginx_status.sh
```
#! /bin/bash

HOST="172.20.100.218"
PORT="8018"
# 监测nginx进程是否存在
function ping {
    /sbin/pidof nginx | wc -l
}
# 监测nginx性能
function active {
    /usr/bin/curl "http://$HOST:$PORT/nginx_status/" 2>/dev/null| grep 'Active' | awk '{print $NF}'
}
function reading {
    /usr/bin/curl "http://$HOST:$PORT/nginx_status/" 2>/dev/null| grep 'Reading' | awk '{print $2}'
}
function writing {
    /usr/bin/curl "http://$HOST:$PORT/nginx_status/" 2>/dev/null| grep 'Writing' | awk '{print $4}'
}
function waiting {
    /usr/bin/curl "http://$HOST:$PORT/nginx_status/" 2>/dev/null| grep 'Waiting' | awk '{print $6}'
}
function accepts {
    /usr/bin/curl "http://$HOST:$PORT/nginx_status/" 2>/dev/null| awk NR==3 | awk '{print $1}'
}
function handled {
    /usr/bin/curl "http://$HOST:$PORT/nginx_status/" 2>/dev/null| awk NR==3 | awk '{print $2}'
}
function requests {
    /usr/bin/curl "http://$HOST:$PORT/nginx_status/" 2>/dev/null| awk NR==3 | awk '{print $3}'
}

$1
```
脚本写完先在本地测试一下
```
chmod +x nginx_status.sh
sh /etc/zabbix/nginx_status.sh active

```
配置/etc/zabbix/zabbix_agentd.conf
```
UnsafeUserParameters=1
```
在脚本中添加定义以下各项的键值,新建nginx_status.conf，输入以下内容
```
more /etc/zabbix/zabbix_agentd.d/nginx_status.conf
UserParameter=nginx.status[*],/etc/zabbix/nginx_status.sh $1
```
zabbix服务端使用zabbix_get 测试获取数据
```
zabbix_get -s 172.20.100.218 -k nginx.status[active]
```

服务端测试正常后就可以在web界面配置监控了，

zabbix的配置流程大致如下：
创建主机组 -》添加主机 -》 创建监控模板 -》 创建应用集 -》创建监控项 -》 创建图像—》创建触发器 -》 创建事件 -》创建处理动作 -》 创建用户组与用户 -》创建告警方式

配置--模板--创建模板--Template App Nginx--更新
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx1.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx1.png)
--点击该模板的应用集--创建应用集--nginx--更新
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx2.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx2.png)
--点击监控项，创建监控项
```
监控项名称及其对应的键值，下图举一例
nginx-accepts    nginx.status[accepts]
nginx-active     nginx.status[active]
nginx-handled    nginx.status[handled]
nginx-requests   nginx.status[requests]
nginx-ping       nginx.status[ping]
nginx-reading    nginx.status[reading]
nginx-waiting    nginx.status[waiting]
nginx-writing    nginx.status[writing]
```
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx3.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx3.png)
--触发器--创建触发器
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx4.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx4.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx5.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx5.png)
就只给这两个创建触发器，最后效果图：
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx6.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zbx_ngx6.png)
至此完毕！
最后把这个模板添加到相应的主机组就可以了。

**//gafana自定义配置nginx监控**
包括系统监控和性能监控。
系统监控：CPU使用率，可用内存，磁盘情况，网卡流量
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gr_ng_sys.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gr_ng_sys.png)
性能监控：nginx连接数，netstat连接数(服务器的并发连接数)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gra_ng_1.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gra_ng_1.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gra_ng_2.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gra_ng_2.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gra_ng_3.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gra_ng_3.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gra_ng_4.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gra_ng_4.png)
**//系统监控**
效果图
![https://raw.githubusercontent.com/yangzhij/images2/master/sys1.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys1.png)
监控CPU
![https://raw.githubusercontent.com/yangzhij/images2/master/sys2.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys2.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/sys3.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys3.png)
监控内存。这个默认没有，需要先在zabbix界面Template OS Linux模板里创建一个监控项
![https://raw.githubusercontent.com/yangzhij/images2/master/sys4.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys4.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/sys5.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys5.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/sys6.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys6.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/sys7.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys7.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/sys8.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys8.png)
监控磁盘使用情况
![https://raw.githubusercontent.com/yangzhij/images2/master/sys9.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys9.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/sys10.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys10.png)
监控网卡流量
![https://raw.githubusercontent.com/yangzhij/images2/master/sys11.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys11.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/sys12.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys12.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/sys13.png](https://raw.githubusercontent.com/yangzhij/images2/master/sys13.png)
**//性能监控**
Nginx连接数
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_nginx1.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_nginx1.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_nginx2.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_nginx2.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_nginx3.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_nginx3.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_nginx4.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_nginx4.png)
监控服务器并发连接数，需先在配置文件里定义一下，然后重启zabbix-agent
```
[root@elk-node1 zabbix_agentd.d]# more nginx_status.conf 
UserParameter=nginx.status[*],/etc/zabbix/nginx_status.sh $1
UserParameter=netstat.status[*],netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'| grep -i $1 | awk '{print $$2}'
UserParameter=netstat.total,netstat -n | grep tcp | wc -l
[root@elk-node1 zabbix_agentd.d]# pwd
/etc/zabbix/zabbix_agentd.d
[root@elk-node1 zabbix_agentd.d]# 
```
创建应用集netstat,
创建监控项
netstat.status[LAST_ACK]
netstat.status[SYN_RECV]
netstat.status[ESTABLISHED]
netstat.status[FIN_WAIT1]
netstat.status[FIN_WAIT2]
netstat.status[TIME_WAIT]
netstat.total
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_net1.png.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_net1.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_net2.png.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_net2.png)
配置grfana 
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_net3.png.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_net3.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_net4.png.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_net4.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_net5.png.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_net5.png)

**//默认的Mill Bil看着别扭吧,调整的方式如下**
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_Graph.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_Graph.png)
![https://raw.githubusercontent.com/yangzhij/images2/master/gr_PieChart.png](https://raw.githubusercontent.com/yangzhij/images2/master/gr_PieChart.png)















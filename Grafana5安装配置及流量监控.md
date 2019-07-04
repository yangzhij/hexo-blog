---
title: centos7下 Grafana5安装配置
date: 2019-02-12 10:22:20
password: passw0rd
tags:
---
**本文档前提条件：Zabbix监控服务器、客户端都已经部署完成，被监控主机已添加完成，Zabbix监控运行正常**
//安装Grafana 5
```
方式1：官方源直接安装
yum -y install https://dl.grafana.com/oss/release/grafana-5.4.2-1.x86_64.rpm
方式2：使用安装包
yum -y install initscripts fontconfig
wget https://dl.grafana.com/oss/release/grafana-5.4.2-1.x86_64.rpm
rpm -Uvh grafana-5.4.2-1.x86_64.rpm
```
//安装服务端图像呈现组件
```
yum install fontconfig freetype* urw-fonts -y
```
//启动grafana,并设置开机启动
```
systemctl daemon-reload
systemctl start grafana-server            #启动服务
systemctl enable grafana-server.service          #设置开机自启
systemctl status grafana-server            #查看服务是否正常启动
```
//开启防火墙放行3000端口
```
firewall-cmd --add-port=3000/tcp --permanent
firewall-cmd --reload
```
//访问，默认用户名密码：admin/admin，第一次登陆会要求改密码
```
http://172.20.100.219:3000
```
//插件安装
```
安装grafana-zabbix插件
grafana-cli plugins install alexanderzobnin-zabbix-app

安装grafana-clock-panel插件：
grafana-cli plugins install grafana-clock-panel

安装grafana饼图插件
grafana-cli plugins install grafana-piechart-panel
```
//重启grafana服务
```
systemctl restart grafana-server
```
点击进去拉到最后，点击zabbix，点击enable，然后最左侧就可以看到zabbix的图标了。
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gr1.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gr1.png)
下一步开始配置数据源
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gr2.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gr2.png)
Zabbix的URL地址为http://172.20.100.219/zabbix/api_jsonrpc.php
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gr3.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gr3.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gr4.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gr4.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gr5.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gr5.png)
点进去选择
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gr6.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gr6.png)

**自定义监控网卡流量**
//最终效果图：
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gra-eth0-1.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gra-eth0-1.png)
//操作如下：
Dashboards->home->New dashboards->Graph
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gra-eth0-2.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gra-eth0-2.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gra-eth0-3.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gra-eth0-3.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/gra-eth0-4.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/gra-eth0-4.png)







---
title: zabbix监控服务端口
date: 2019-02-11 16:00:58
password: passw0rd
tags:
---
1.被监控端上编写脚本，注意两个脚本属主为zabbix:zabbix
port_check.sh为端口自发现脚本,编写完后添加执行权限chmod +x port_check.sh
port.conf为指定的监控端口号

具体操作
编辑port_check.sh	
```
mkdir /etc/zabbix/script/
cd /etc/zabbix/script/
more port_check.sh
#/bin/bash
CONFIG_FILE=/etc/zabbix/script/port.conf
Check(){
    grep -vE '(^ *#|^$)' ${CONFIG_FILE} | grep -vE '^ *[0-9]+' &> /dev/null
    if [ $? -eq 0 ]
    then
        echo Error: ${CONFIG_FILE} Contains Invalid Port.
        exit 1
    else
        portarray=($(grep -vE '(^ *#|^$)' ${CONFIG_FILE} | grep -E '^ *[0-9]+'))
    fi
}
PortDiscovery(){
    length=${#portarray[@]}
    printf "{\n"
    printf  '\t'"\"data\":["
    for ((i=0;i<$length;i++))
      do
        printf '\n\t\t{'
        printf "\"{#TCP_PORT}\":\"${portarray[$i]}\"}"
        if [ $i -lt $[$length-1] ];then
                    printf ','
        fi
      done
    printf  "\n\t]\n"
    printf "}\n"
}
port(){
    Check
    PortDiscovery
}
port 
```
同目录下编辑port.conf，配置文件port.conf每个端口号一行
```
more port.conf
9551  
9552  
9050  
9052  
7052  
8050  
9053  
5150
```
更改属主
```
cd /etc/zabbix/script/
chown zabbix.zabbix ./*
```
2.修改被监控端的zabbix_agent.conf配置文件
```
vi /etc/zabbix/zabbix_agentd.conf
UserParameter=port.alert,/etc/zabbix/script/port_check.sh
```
3.重启agent端zabbix服务
```
ystemctl restart zabbix-agent
```
4.服务端用zabbix-get看能否获取数据:
```
[root@elk-node1 dejavu]#  zabbix_get -s 172.20.100.128 -k port.alert
{
        "data":[
                {"{#TCP_PORT}":"9551"},
                {"{#TCP_PORT}":"9552"},
                {"{#TCP_PORT}":"9050"},
                {"{#TCP_PORT}":"9052"},
                {"{#TCP_PORT}":"7052"},
                {"{#TCP_PORT}":"8050"},
                {"{#TCP_PORT}":"9053"},
                {"{#TCP_PORT}":"5150"}
        ]
}
```
5.接下来就在zabbix管理web添加自动发现规则：配置在默认的模板Template OS Linux里即可
添加自动发现规则：
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort1.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort1.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort2.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort2.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort3.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort3.png)
之后点击刚创建的自动发现规则tcp port discover，开始添加监控项原型
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort4.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort4.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort5.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort5.png)
完毕之后，添加触发器报警：值为0则端口不通，值为1则端口通
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort6.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort6.png)
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort7.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort7.png)
完成之后，可以再弄个图形原型
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort8.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort8.png)
添加完之后等一会儿图就会生成了，如下图所示。然后可以测试这关闭几个进程来验证。
![https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort9.png](https://raw.githubusercontent.com/yangzhij/myself-images/master/zaPort9.png)














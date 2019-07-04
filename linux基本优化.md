---
title: linux基本优化
date: 2019-05-23 09:55:48
password: passw0rd
tags:
---
//内核优化(调整Linux网络内核对TCP连接的有关限制等)
```
vim /etc/sysctl.conf
net.ipv4.route.gc_timeout = 100
net.core.rmem_default = 256960
net.ipv4.ip_local_port_range = 1024 65000
net.core.rmem_max = 513920
net.core.wmem_default = 256960
net.core.wmem_max = 513920
net.ipv4.tcp_rmem = 8760  256960  4088000
net.ipv4.tcp_rmem = 8760  256960  4088000
net.ipv4.tcp_wmem = 8760  256960  4088000
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_sack = 1
net.ipv4.tcp_fack = 1
net.ipv4.tcp_congestion_control = htcp
net.ipv4.tcp_mtu_probing = 1
net.core.optmem_max = 81920
net.core.netdev_max_backlog = 2000
net.ipv4.tcp_no_metrics_save = 1
net.core.somaxconn = 262144
net.ipv4.tcp_mem = 131072  262144  524288
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 262144
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
kernel.shmmax = 134217728
net.ipv4.tcp_keepalive_time = 1800
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 3
//参数及时生效命令
sysctl -p   
```

//操作系统对一个进程打开的文件句柄数量的限制
```
vi /etc/security/limits.conf 
*                soft    nofile  65535
*                hard    nofile  65535
*                soft    nproc   65535
*                hard    nproc   65535
```
//文件末尾添加
```
vi /etc/sysctl.conf
fs.file-max=65535   #保存退出
sysctl -p  生效配置
```

//设置linux用户的最大进程数以及每个进程可打开的文件数
```
ulimit -n 65535
ulimit -u 65535
```

//历史数据条数优化
```
vi /etc/profile
# add at 2019-05-13
HISTFILESIZE=30000
HISTSIZE=10000
HISTTIMEFORMAT='%Y-%m-%dT%T '
export HISTTIMEFORMAT
```



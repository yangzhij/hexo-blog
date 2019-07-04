---
title: MYSQL压测 sysbench
date: 2019-06-03 16:02:22
tags:
---
sysbench安装
```
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum install sysbench
```

测试准备: 先创建一个库sbtest, 然后通过下面的语句来生成10张表 每个表填充100W条数据
```
sysbench /usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --mysql-host=172.20.100.115 --mysql-port=3306 \
 --mysql-user=root --mysql-password=123456 \
 --oltp-num-tables=10 --oltp-table-size=1000000 --oltp-test-mode=complex  --report-interval=10 prepare

//释义
--oltp-num-tables=10   生成 10 张测试表
--oltp-table-size=1000000	每张表填充数据为1000000
--oltp-test-mode=complex	执行模式
--report-interval=10   表示每10秒输出一次测试进度报告
```

开始测试
```
sysbench /usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --mysql-host=172.20.100.115 --mysql-port=3306 \
 --mysql-user=root --mysql-password=123456 \
 --oltp-num-tables=10 --oltp-table-size=1000000 --oltp-test-mode=complex \
 --num-threads=128 --max-time=120 --report-interval=10 run

//释义
--max-time=120    压测持续时间，此为2分钟
--num-threads=128 并发线程数，可以理解为模拟的客户端并发连接数（可以把num-threads依次递增（16,36,72,128,256,512））
```

//mysql自带压测工具mysqlslap
```
#80个客户端插入100W条数据需要多长时间？
mysqlslap -uroot -p123456 -a --auto-generate-sql-write-number=1 -c 80 -i 1 --auto-generate-sql-load-type=write --number-of-queries=1000000
```
//释义
```
-a 自动生成测试表和数据，表示用mysqlslap工具自己生成的SQL脚本来测试并发压力。
--auto-generate-sql-write-number=1   用系统自己生成的SQL脚本来测试,初始化1行
-c 80 表示并发量，也就是模拟80个客户端同时执行write
-i 1 要运行这些测试多少次
--auto-generate-sql-load-type=write   要测试的是读还是写还是两者混合的（read,write,update,mixed）
--number-of-queries=100000        总共运行这么多次写入

//上述命令输出内容
mysqlslap: [Warning] Using a password on the command line interface can be insecure.
Benchmark
        Average number of seconds to run all queries: 109.042 seconds   运行所有查询的平均秒数：109.042秒
        Minimum number of seconds to run all queries: 109.042 seconds	运行所有查询的最小秒数：109.042秒
        Maximum number of seconds to run all queries: 109.042 seconds	运行所有查询的最大秒数：109.042秒
        Number of clients running queries: 80	运行查询的客户端数：80
        Average number of queries per client: 12500		每个客户端的平均查询数：12500
```
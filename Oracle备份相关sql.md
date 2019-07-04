---
title: Oracle备份相关sql
date: 2019-01-06 16:56:18
tags:
---
```
select * from dba_directories;   --查看数据库备份路径
--Oracle备份文件是以dmp结尾，这种文件是oracle的逻辑备份文件

grant create any directory to scott;   --给scott用户创建目录的权限

create directory dmp(目录名随意)  as '/data/dmp';  --创建备份目(linux)，create不进行判断，若存在，则报错说对象已存在。

create or replace directory dmp as '/data/dmp'; --创建(备份)目录，若已存在，则替代，没有则创建.

create directory dmp as 'D:\temp\dmp';   --创建备份目录(Windows)
```

数据备份,通过df –lh查看剩余空间，
空间不够的话可以将历史备份数据挪走或者删除上次备份的数据。 
然后看看归档日志是否需要清理。 最后进行备份。   

```
expdp domain/domain TABLES=dn_whois  dumpfile=dn_whois.dmp logfile=dn_whois.log directory=dmp 
impdp bims/bims123  dumpfile=dn_whois.dmp logfile=dn_whois201803.log  directory=dmp  REMAP_SCHEMA=domain:bims


expdp 用户名/密码@库名(数据库实例名称)

remap_schema当你从A用户导出的数据，想要导入到B用户中去，就使用这个：remap_schema=A:B


expdp 用户名/密码 [tables = 表名] directory指定备份目录  dmpfile备份文件名称   logfile日志文件名称  schema数据库对象


expdp ngoss2/asp_186tc2we 
directory=dmp dumpfile=ngoss20180126_%U.dmp logfile=ngoss20180126.log 
schemas=ngoss2,terminal,domain,ns     --用户名（导出用户及其对象） 
parallel=8        --使用并行模式来执行当前sql，按理是数字越大，执行效率越高，但要考虑服务器CPU个数
cluster=no        --只在单实例上导出。只让它备份到一台服务器上。


命令行输入：lscpu
Core(s) per socket:    8            #每个cpu，有8个核
Socket(s):             2      #总共有2个cpu  



expdp henan/henan@orcl    ---用户名/密码@库名
dumpfile=henan20170825_%U.dmp    --备份文件
logfile=henan20170825.log    --备份日志
schemas=henan    --用户名（导出用户及其对象）
parallel=4    --并行执行   4个文件  
CLUSTER=N    -单点备份
directory=hndb_bak    --备份目录
EXCLUDE=TABLE:\"IN\(\'T_PERF_RTT\',\'T_PERF_WEBRTT\'\)\"        --导出不包含T_PERF_RTT, T_PERF_WEBRTT  的其它所有对象.



expdp username/password 
dumpfile=$EXPFILE 
logfile=$EXPLOG 
directory=expdir 
exclude=TABLE:\"LIKE \'MV%\'\"    --排除以MV开头的表



expdp username/password 
dumpfile=$EXPFILE 
logfile=$EXPLOG 
directory=expdir 
exclude=TABLE:\"LIKE \'MV%\'\", exclude=TABLE:\"LIKE \'TMP%\'\"     --排除以MV开头的表和以TMP开头的表
```


A库的数据导入到B库中？？？？？

如果数据库B中已经存在了需要导入的用户且用户默认表空间不一致，那么需要指定表空间和指定用户导入。

https://jingyan.baidu.com/article/a3a3f811e8492c8da2eb8aad.html

group by     分组
having	筛选
select 
distinct     去重
order by     排序
limit     限制条数

Not null ---不可空      null----可空  
Default  默认值
not null+unique:不为空且唯一  =======》 primary key主键

inner join内连接，只连接匹配的行（找两张表共有的部分）
外链接之左连接：优先显示左表全部记录。以左表为准。


数据库备份导入sql
```
1. 先创建备份路径：
create or replace directory dmp as '/data/dmp';
2. 整表备份：
expdp ns/netspeed@netspeed schemas=ns,ls directory=data_bak_20140609 dumpfile =ns_20140609.dmp logfile=ns_20140609.log 
3. 整表恢复：
impdp jiangsu/jiangsu directory=data_bak_20140609 dumpfile=20140604.dmp logfile=20140604.log  REMAP_SCHEMA=terminal:jiangsu (原表用户名---        目的表用户名)
4.单表备份：
expdp ns/netspeed tables=ns_monitorpoint_whwstar dumpfile=whw_20140609.dmp logfile=whw_20140609.log directory=data_bak_20140609
5. 单表恢复：
impdp ns/netspeed directory=data_bak_20140609 dumpfile=whw_20140609.dmp logfile=whw_20140609.dmp.log  tables=ns_monitorpoint_whwstar REMAP_SCHEMA=ns:ns

--河南管局数据恢复
impdp jiangsu/jiangsu     
directory=dmp 
dumpfile=jiangsu20141029.dmp 
logfile=jiangsu20141029.log 
tables=t_perf_webrtt20141029pxb,t_perf_rtt20141029pxb,t_perf_dnsrtt20141029pxb 
REMAP_SCHEMA=jiangsu:jiangsu 
remap_tablespace='(JIANGSU_TBSPACE1:JIANGSU_TBSPACE1,HENAN_TBSPACE1:JIANGSU_TBSPACE1,NGOSS_TBSPACE2:JIANGSU_TBSPACE1,TERMINAL_TBSPACE1:JIANGSU_TBSPACE1,JIANGSU_TBSPACE:JIANGSU_TBSPACE1)'

--expdp henan/henan tables=ZXX2 dumpfile=ZXX2_20141029.dmp logfile=ZXX2_20141029.log directory=dmp

--impdp henan/henan directory=dmp dumpfile=ZXX2_20141029.dmp logfile=ZXX2_20141029.log  tables=ZXX2  REMAP_SCHEMA=ngoss2:henan


exp userid=ngoss2/asp_186tc2we full=y file=exp20130923.dmp log=exp20141117.log tables=t_web_struc

导入陕西库
impdp shanxi/shanxi    --当前库的用户名密码（有多个用户名的话具体看是要导入到哪个用户下）
directory=dmp 
dumpfile=jiangsu20140926.dmp 
logfile=impshanxi.log    
REMAP_SCHEMA=jiangsu:shanxi     --当你从A用户导出的数据,想要导入到B用户中去,就使用这个:remap_schema=A:B 
REMAP_TABLESPACE=JIANGSU_TBSPACE:SHANXI_TBSPACE1,TERMINAL_TBSPACE1:SHANXI_TBSPACE1,HENAN_TBSPACE1:SHANXI_TBSPACE1  

--remap_tablespace 与上面类似，数据库对象本来存在于tbs_a表空间，现在你不想放那儿了，想换到tbs_b，就用这个remap_tablespace=tbs_a:tbs_b  结果是所有tbs_a中的对象都会建在tbs_b表空间中。搜索
--这样做的前提是目标用户B和目标表空间tbs_b存在


四川库部分表导入
impdp sichuan/sichuan 
directory=dmp 
dumpfile=sichuan20151226.dmp 
logfile=sichuan20151227.log    
REMAP_SCHEMA=sichuan:sichuan 
REMAP_TABLESPACE=JIANGSU_TBSPACE:SICHUAN_TBSPACE1,TERMINAL_TBSPACE1:SICHUAN_TBSPACE1,HENAN_TBSPACE1:SICHUAN_TBSPACE1,SICHUAN_TBSPACE1:SICHUAN_TBSPACE1

---新建按天分区语句
--时延表按天分区
create table T_PERF_RTT
(
  ID             NUMBER not null,
  MPID           NUMBER,
  DBTIME         DATE,
  PERIOD         NUMBER,
  CMOIID         VARCHAR2(32),
  VCMOINAME      VARCHAR2(255),
  PERFID         NUMBER,
  SYSTIME        DATE,
  DTIME          DATE,
  HOP            NUMBER,
  P1             NUMBER,
  P2             NUMBER,
  P3             NUMBER,
  MTU            NUMBER,
  MEID           NUMBER,
  IPID           NUMBER,
  IP             VARCHAR2(64),
  IMISSIONID     NUMBER,
  IMISSIONTYPEID NUMBER,
  IMISSIONFLAG   NUMBER,
  IDEALFLAG      NUMBER default 0
)
partition by range (DTIME)
INTERVAL (NUMTODSINTERVAL(1,’day’))(PARTITION p_first VALUES LESS THAN (to_date('2010-08-01','yyyy-mm-dd')));

alter table T_PERF_RTT
  add constraint PK_T_PERF_RTT primary key (ID);
-- Create/Recreate indexes 
create index IDX_T_PERF_RTT_1 on T_PERF_RTT (DTIME, MPID, MEID);
create index IDX_T_PERF_RTT_2 on T_PERF_RTT (MPID, MEID, IP);
create index IDX_T_PERF_RTT_3 on T_PERF_RTT (MPID, IP);
create index IDX_T_PERF_RTT_4 on T_PERF_RTT (DTIME);
create index IDX_T_PERF_RTT_5 on T_PERF_RTT (MPID, MEID, IP, DTIME);

```
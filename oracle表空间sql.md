---
title: oracle表空间sql
date: 2019-01-06 15:43:12
tags:
---

--查询数据库表空间位置
```
select name from v$datafile; 
```
--创建表空间（根据系统表空间规则创建即可，修改文件名称就可以）
```
create tablespace ts_test  datafile '+DATA/orcl/datafile/ts_test1.dbf' size 100m ;
```
--创建用户 zahngsan并设置默认用户表空间  
``` 
create user  zahngsan(用户名) identified by  zahngsan#1234(密码) default tablespace ts_test; 
```
--授权dba权限给 zahngsan
```
grant connect,resource,dba to  zahngsan;      
--connect：拥有Connect权限的用户只可以登录Oracle，不可以创建实体，不可以创建数据库结构。
--resource:拥有Resource权限的用户只可以创建实体，不可以创建数据库结构。
--dba：拥有全部特权，是系统最高权限，只有DBA才可以创建数据库结构。
```
--扩充表空间
```
alter tablespace ts_test add datafile '+DATA/orcl/datafile/ts_test2.dbf' size 1G  --给表空间增加数据文件   
```
--给表空间增加数据文件时，也可更换磁盘，即目录。（并设置数据文件自增）
```
alter tablespace ts_test add datafile '/data/da_orac/datafile/ts_test2.dbf' size 1G （autoextend ON next 1G maxsize 100G; --允许当前数据文件自动扩展 ） 
```
--创建用户  （创建用户后系统会默认给该用户分配一个表空间users）
```
create user zzg identified by zzg123;  
```
--修改用户密码
```
alter user zzg identified by unis;  
```
--将创建好的表空间分配给用户
```
alter user zzg default tablespace ts_zzg;  
```
--查看所有用户所在的表空间
```
select username,default_tablespace from dba_users;  
```
--删除用户及相关对象
```
drop user  zahngsan cascade;
--提示：无法删除当前已链接的用户

--查看用户的连接状况以及session的状态。
--（status 为要删除用户的session状态，如果还为inactive，说明没有被kill掉，如果状态为killed，说明已kill。）

select saddr,sid,serial#,paddr,username,status from v$session where username=' zahngsan'

--在命令窗口杀掉sid,serial#
alter system kill session'771,31373';

--去除之后重新执行
drop user  zahngsan cascade;
```

--创建表空间  --每次扩展10M，最高扩展至200M
```
create tablespace DATATEST datafile 'D:\DATABASE\10.2.0\ORCL\DATATEST.dbf' size 100M autoextend ON next 10M maxsize 200M;   
--on next 10M maxsize unlimited   --每次扩展10M，无限制扩展
```
--创建临时表空间
```
create temporary tablespace DATATEST_TEMP tempfile 'D:\DATABASE\10.2.0\ORCL\DATATEST_TEMP.dbf' size 50M autoextend ON next 10M maxsize 100M; 
```

--删除表空间
```
drop tablespace xxx 
```
--删除有任何数据对象的表空间
```
drop tablespace xxx including contents and datafiles;
```
一个数据库可以有多个用户。 但一个用户下的表数据不会到另一个用户。
从oracle10g开始，可以对除SYSTEM和SYSAUX之外的表空间进行重命名。
```
语法：alter tablespace oldname(旧名称) rename to newname(新名称);
```

表空间查询G
```
--1T=1024G
--1G=1024MB
--1M=1024KB
--1K=1024Bytes
--1M=11048576Bytes
--1G=1024*11048576Bytes=11313741824Bytes
SELECT a.tablespace_name "表空间名",
--total / (1024 * 1024 * 1024) "表空间大小(G)", 
round(total / (1024 * 1024 * 1024),3) "表空间大小(G)", --保留三位小数
round(free / (1024 * 1024 * 1024),3) "表空间剩余大小(G)",
round((total - free) / (1024 * 1024 * 1024),3) "表空间使用大小(G)",
round((total - free) / total, 4) * 100 "使用率 %"
FROM (SELECT tablespace_name, SUM(bytes) free
FROM dba_free_space
GROUP BY tablespace_name) a,
(SELECT tablespace_name, SUM(bytes) total
FROM dba_data_files
GROUP BY tablespace_name) b
WHERE a.tablespace_name = b.tablespace_name 


大唐--表空间查询M
SELECT D.TABLESPACE_NAME,
       SPACE "SUM_SPACE(M)", 
     /*  BLOCKS SUM_BLOCKS,*\*/
       SPACE - NVL(FREE_SPACE, 0) "USED_SPACE(M)",           
       FREE_SPACE "FREE_SPACE(M)",
       ROUND((1 - NVL(FREE_SPACE, 0) / SPACE) * 100, 2) "USED_RATE(%)"
  FROM (SELECT TABLESPACE_NAME,              
               ROUND(SUM(BYTES) / (1024 * 1024), 2) SPACE,
               SUM(BLOCKS) BLOCKS       
          FROM DBA_DATA_FILES        
         GROUP BY TABLESPACE_NAME) D,       
       (SELECT TABLESPACE_NAME,
               ROUND(SUM(BYTES) / (1024 * 1024), 2) FREE_SPACE       
          FROM DBA_FREE_SPACE
         GROUP BY TABLESPACE_NAME) F
WHERE D.TABLESPACE_NAME = F.TABLESPACE_NAME(+)
order by "USED_RATE(%)" desc;   



```



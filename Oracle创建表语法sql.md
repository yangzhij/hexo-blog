---
title: Oracle创建表语法sql
date: 2019-01-06 17:04:58
tags:
---
字符串----  varchar2(20)
数字---------number
日期--------date

--创建表--IP段
```
create table T_IPIP_SEG
(
  ipstart      VARCHAR2(64) not null,
  ipend   VARCHAR2(64) not null,
  prefix       VARCHAR2(32),
  prefixlth    NUMBER,
  type         NUMBER,
  operid       VARCHAR2(16),
  asnum        NUMBER,
  cityid       NUMBER,
  dtime        DATE,
  lng          NUMBER,
  lat          NUMBER,
  info         VARCHAR2(50),
  compresstime NUMBER,
  mocid        NUMBER,
  location     VARCHAR2(128),
  resid        NUMBER,
  vcdesc       VARCHAR2(256)
);
```
--建立索引
```
create unique index IDX_T_IPIP_SEG1 on T_IPIP_SEG(IPSTART,IPEND);
```
--索引就是对某一列进行排序
create unique index 
--是创建唯一索引，但前提是列上的数据不能有重复值。 保证索引列的取值是唯一的。
create index
--是创建普通索引



--创建视图
create view 视图名 as select *，*，*... from 表名 where 

比如一个比较复杂的查询不想每次都写很多语句，就可以写个视图。
把复杂的查询SQL写到视图里，下次查询的时候只需要使用select * from 视图名就可以了  
或者给特定用户开放某些表的读取权限，但要加一些行和列的限制，也可以写个视图。              

oracle中，删除基表后，建立在此表上的视图仍然保留，但不能用，还需要用删除视图语句删除。

删除视图，对基表没有影响。

删除视图：  drop view 视图名
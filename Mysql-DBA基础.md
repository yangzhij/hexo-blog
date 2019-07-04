---
title: Mysql-DBA基础
date: 2019-05-31 01:16:37
tags:
---
### //数据库相关概念
DBMS：数据库管理系统，用于管理DB实例
DB：数据库，保存一组有组织的数据的容器
SQL: 结构化查询语言
### //数据库存储数据特点
数据在表中，表在库中。一个数据库中可以有多个表，表名具有唯一性。表由列组成，也称列为字段。表中的数据是按行存储的。
### //SQL常见命令
```
show databases;		查看所有的数据库
use 库名	打开指定的库
show tables;	显示库中的所有表
show tables from 库名;   显示指定库中的所有表
desc 表名;    查看表结构
select version();    查看mysql版本
mysql --version 或 mysql --V
```
### //语法规范
```
不区分大小写，命令最好用分号结尾。
注释：
单行注释：#注释文字
单行注释：-- 注释文字，--后面有一个空格
多行注释：/* 注释文字 */
```
### //SQL语言分类
1.DML:数据操作语言
select（DQL 数据查询语言）、insert 、update、delete
2.DDL：数据定义语言，针对的是库和表
create、drop、alter
3.TCL：事务控制语言
commit、rollback
4.DCL：数据权限控制语言
### //DQL语言
select查询完的结果，是一个虚拟的表格，不是真实存在。要查询的东西 可以是常量值、可以是表达式、可以是字段、可以是函数
1.基础查询
```
select 查询列表 from 表名;
select distinct  列名 from 表名;   distinct去重
select concat(‘参数1’,'参数2',...) AS '别名' from 表名;    concat拼接，AS别名
select ifnull(列名,0) AS 别名 from 表名;     infull判断字段是否为空，是的话则返回0，不是返回原本的值
```
2.条件查询
```
select 查询列表 from 表名 where 筛选条件;
```
筛选条件分类：
```
按条件表达式
	条件运算符：> < >= <= = != <>
按逻辑表达式
	逻辑运算符：and（&&）or(||) not(!)
	逻辑运算符的作用： 用于连接条件表达式
		select * from 表名 where not(条件1  and 条件2) or 条件3;
模糊查询
	like
		like '%a%'   
		like '__n_w%'  第三个字符为n,第五个字符为w
		like '_\_%';   第二个字符为_，其中\为转义;	like '_$_%' escape '$';  转移符可以是任意的，后面用escape定义一下就好了
范围查询
	between and
		如id在100到120之间   id >= 100 and id <=120 可以写成 id between 100 and 120;
	in 判断某字段的值是否属于in列表的某一项
		... where id in (15,16,17);
空判断		
	is null   为空
	is not null 不为空

3.排序查询 order by 
语法： SELECT * FROM 表名 ORDER BY 列1 ASC|DESC [,列2 ASC|DESC,...]

ASC 默认升序
DESC 降序


4.聚合查询
COUNT(*)  总数
MAX(列名) 最大值
MIN(列名) 最小值
SUM(列名) 求和
AVG(列名) 平均值


5.分组  group by

GROUP BY + GROUP_CONCAT() GROUP_CONCAT(字段名)可以作为一个输出字段来使用
SELECT gender,GROUP_CONCAT(NAME) FROM students GROUP BY gender;

GROUP BY + 聚合函数 
SELECT gender,AVG(height) FROM students GROUP BY gender;

GROUP BY + HAVING having表达式，用来分组以后设定条件筛选数据，功能和where一样，但是having只能用于group BY


6.子查询
在一个select语句中，嵌入另外一个select语句，被嵌入的语句就是子查询语句
主查询 主要查询的对象,第一条 SELECT 语句

主查询和子查询的关系

子查询是嵌入到主查询中
子查询是辅助主查询的,要么充当条件,要么充当数据源
子查询是可以独立存在的语句,是一条完整的 SELECT 语句

7.分页
当数据量很大的时候，就不可能在一页中查看所有数据了，需要对它进行分页操作
语法: SELECT * FROM 表名 LIMIT START,COUNT
举例：select * FROM WHERE gender=1 LIMIT 0,3; 查询前三条男生记录



8. mysql 权限
超级用户root
库级用户
表级用户 
字段(某些列)级别用户
存储程序级别用户



9. 连接查询
# 语法： select * from 表1 inner或left或right join 表2 on 表1.列 = 表2.列
内连接
左连接
右连接

#内连接 查询的结果为两个表匹配到的数据,并不是两张表所有的数据

SELECT * FROM students INNER JOIN classes ON students.cls_id = classe.id;

#使用左关联还是右关联  主要看侧重于哪张表

SELECT * FROM students AS s LEFT JOIN classes AS c ON s.cls_id = c.id; 

SELECT * FROM students AS s RIGHT JOIN classes AS c ON s.cls_id = c.id; 

SELECT s.name,c.name FROM students s INNER JOIN classes c ON s.`cls_id` = c.`id`; 

```

































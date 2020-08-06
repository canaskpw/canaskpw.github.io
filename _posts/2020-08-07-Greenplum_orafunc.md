---
title: Greenplum实现Oracle兼容函数
#description: 
date: 2020-08-07 02:50:00
categories: 
 - greenplum
tags:
 - greenplum
---

## 摘要
近期在做一个系统由Oracle到Greenplum上的迁移，Oracle存储过程逻辑中使用的部分函数在Greenplum没有，由于Oracle不开源只能自定义GP函数来实现同样的逻辑，一个个自定义函数工程量比较大，在阅读官方文档时发现有通过实现orafunc实现Oracle兼容函数的功能。遂在PC上搭建Greenplum集群环境实现orafunc兼容函数后在开发、生产环境安装，减小工作量，提升开发效率。
好记性不如烂笔头，作此笔记

<!-- more -->

# Greenplum实现Oracle兼容函数

## 查询orafunc.sql路径
```shell
[gpadmin@mdw gpseg-1]$ find / -name orafunc*
/usr/local/greenplum-db-5.23.0/share/postgresql/contrib/orafunc.sql
```

## 安装orafunc.sql
```shell
#安装命令
[gpadmin@mdw gpseg-1]$ psql -d hhb_db -f $GPHOME/share/postgresql/contrib/orafunc.sql

#安装过程
CREATE SCHEMA
SET
BEGIN
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
psql:/usr/local/greenplum-db/./share/postgresql/contrib/orafunc.sql:172: NOTICE:  aggregate oracompat.listagg(text) does not exist, skipping
DROP AGGREGATE
CREATE AGGREGATE
psql:/usr/local/greenplum-db/./share/postgresql/contrib/orafunc.sql:178: NOTICE:  aggregate oracompat.listagg(text,text) does not exist, skipping
DROP AGGREGATE
CREATE AGGREGATE
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
COMMIT

```

## 查看已安装的Oracle兼容函数
```shell
[gpadmin@mdw gpseg-1]$ psql -d hhb_db
psql (8.3.23)
Type "help" for help.

hhb_db=# \df oracompat.*
                                                 List of functions
  Schema   |       Name       |     Result data type     |               Argument data types               |  Type  
-----------+------------------+--------------------------+-------------------------------------------------+--------
 oracompat | add_months       | date                     | day date, value integer                         | normal
 oracompat | bitand           | bigint                   | bigint, bigint                                  | normal
 oracompat | concat           | text                     | anyarray, anyarray                              | normal
 oracompat | concat           | text                     | anyarray, text                                  | normal
 oracompat | concat           | text                     | text, anyarray                                  | normal
 oracompat | concat           | text                     | text, text                                      | normal
 oracompat | dump             | character varying        | "any"                                           | normal
 oracompat | dump             | character varying        | "any", integer                                  | normal
 oracompat | instr            | integer                  | str text, patt text                             | normal
 oracompat | instr            | integer                  | str text, patt text, start integer              | normal
 oracompat | instr            | integer                  | str text, patt text, start integer, nth integer | normal
 oracompat | last_day         | date                     | value date                                      | normal
 oracompat | listagg          | text                     | text                                            | agg
 oracompat | listagg          | text                     | text, text                                      | agg
 oracompat | listagg1_transfn | text                     | text, text                                      | normal
 oracompat | listagg2_transfn | text                     | text, text, text                                | normal
 oracompat | lnnvl            | boolean                  | boolean                                         | normal
 oracompat | months_between   | numeric                  | date1 date, date2 date                          | normal
 oracompat | nanvl            | double precision         | double precision, double precision              | normal
 oracompat | nanvl            | numeric                  | numeric, numeric                                | normal
 oracompat | nanvl            | real                     | real, real                                      | normal
 oracompat | next_day         | date                     | value date, weekday integer                     | normal
 oracompat | next_day         | date                     | value date, weekday text                        | normal
 oracompat | nlssort          | bytea                    | text, text                                      | normal
 oracompat | nvl              | anyelement               | anyelement, anyelement                          | normal
 oracompat | nvl2             | anyelement               | anyelement, anyelement, anyelement              | normal
 oracompat | reverse          | text                     | str text                                        | normal
 oracompat | reverse          | text                     | str text, start integer                         | normal
 oracompat | reverse          | text                     | str text, start integer, _end integer           | normal
 oracompat | round            | date                     | value date                                      | normal
 oracompat | round            | date                     | value date, fmt text                            | normal
 oracompat | round            | timestamp with time zone | value timestamp with time zone                  | normal
 oracompat | round            | timestamp with time zone | value timestamp with time zone, fmt text        | normal
 oracompat | substr           | text                     | str text, start integer                         | normal
 oracompat | substr           | text                     | str text, start integer, len integer            | normal
 oracompat | trunc            | date                     | value date                                      | normal
 oracompat | trunc            | date                     | value date, fmt text                            | normal
 oracompat | trunc            | timestamp with time zone | value timestamp with time zone                  | normal
 oracompat | trunc            | timestamp with time zone | value timestamp with time zone, fmt text        | normal
(39 rows)
```
## 增加数据库对orafun搜索路径
Oracle的兼容函数都安装在oracompat的schema下面。我们想使用oracompat下的orafun，需要在函数前显式加oracompat模式名或者修改数据库的搜索路径：
```sql
hhb_db=# select oracompat.nvl(null,65535);
  nvl  
-------
 65535
(1 row)
```
修改数据库的搜索路径：
```sql
hhb_db=# ALTER DATABASE hhb_db SET search_path = public, oracompat;
ALTER DATABASE

hhb_db=# select add_months('20200430',1);
 add_months 
------------
 2020-05-31
(1 row)

```
ALTER DATABASE

## orafun给普通用户赋权
orafunc 是使用gpadmin 超级管理员权限赋权，普通用户hhb权限不够无法访问，我们需要将oracompat的权限赋予hhb用户
```sql
--使用gpadmin用户登录
hhb_db=# grant all on schema oracompat to hhb;
GRANT

--使用hhb用户登录实验
hhb_db=# select last_day('20200807');
  last_day  
------------
 2020-08-31
(1 row)
```
> 日常开发中切记最小赋权，不可图方便直接使用管理员权限赋权或者进行操作
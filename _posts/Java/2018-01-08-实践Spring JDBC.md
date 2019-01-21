---
layout: post
title: "实践Spring JDBC"
date: 2018-01-08 14:05:00 +0800
categories: Java
tags: database jdbc java spring-jdbc
---

Spring JDBC是Spring对数据库访问的初级封装

## *Template

```
JdbcOperations
```

### JdbcTemplate

`JdbcTemplate`主要包含如下方法：

1. execute()：一般用于执行DDL
2. update() and batchUpdate()，一般用于执行新增、删除和修改数据
3. query() and queryFor*()，一般用于执行查询数据
4. call()，一般用于执行存储过程和函数

KeyHolder

RowMapper

ResultSetExtractor

PreparedStatementSetter

PreparedStatementCreator

RowCallbackHandler

```sql
delete from BookInfo where bid =?
```



### NamedParameterJdbcTemplate

基于`JdbcTempate`封装从而支持命名参数特性

#### SqlParameterSource

```sql
delete from mb_mcht_info t where t.mcht_name = :mchtName
update mb_mcht_info t set t.mcht_address = :mchtAddress where t.mcht_name = :mchtName
select t.* from mb_mcht_info t where t.mcht_name = :mchtName
```



## *DaoSupport

### JdbcDaoSupport

### NamedParameterJdbcDaoSupport

```sql

```



## 异常体系

## 声明式事物
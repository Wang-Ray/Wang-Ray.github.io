---
layout: post
title: "实践MySQL之schema之information_schema"
categories: 数据库
tags: database MySQL
---

### schemata

　schemata表：这个表里面主要是存储在mysql中的所有的数据库的信息

### tables

　　tables表：这个表里存储了所有数据库中的表的信息，包括每个表有多少个列等信息。

```mysql
create table information_schema.TABLES
(
    TABLE_CATALOG   VARCHAR  not null,
    TABLE_SCHEMA    VARCHAR  not null,
    TABLE_NAME      VARCHAR  not null,
    TABLE_TYPE      VARCHAR  not null,
    ENGINE          VARCHAR  not null,
    VERSION         BIGINT   not null,
    ROW_FORMAT      VARCHAR  not null,
    TABLE_ROWS      BIGINT   not null,
    AVG_ROW_LENGTH  BIGINT   not null,
    DATA_LENGTH     BIGINT   not null,
    MAX_DATA_LENGTH BIGINT   not null,
    INDEX_LENGTH    BIGINT   not null,
    DATA_FREE       BIGINT   not null,
    AUTO_INCREMENT  BIGINT   not null,
    CREATE_TIME     DATETIME not null,
    UPDATE_TIME     DATETIME not null,
    CHECK_TIME      DATETIME not null,
    TABLE_COLLATION VARCHAR  not null,
    CHECKSUM        BIGINT   not null,
    CREATE_OPTIONS  VARCHAR  not null,
    TABLE_COMMENT   VARCHAR  not null,
    AUTO_PARTITION  VARCHAR  not null
)
    engine = MEMORY
    charset = utf8;
```

### columns

　　columns表：这个表存储了所有表中的表字段信息。

### statistics

　　statistics表：存储了表中索引的信息。

### user_privileges

　　user_privileges表：存储了用户的权限信息。

### schema_privileges

　　schema_privileges表：存储了数据库权限。

### table_privileges

　　table_privileges表：存储了表的权限。

### column_privileges

　　column_privileges表：存储了列的权限信息。

### character_sets

　　character_sets表：存储了mysql可以用的字符集的信息。

### collations

　　collations表：提供各个字符集的对照信息。

### collation_character_set_applicability

　　collation_character_set_applicability表：相当于collations表和character_sets表的前两个字段的一个对比，记录了字符集之间的对照信息。

### table_constraints

　　table_constraints表：这个表主要是用于记录表的描述存在约束的表和约束类型。

### key_column_usage

　　key_column_usage表：记录具有约束的列。

### routines

　　routines表：记录了存储过程和函数的信息，不包含自定义的过程或函数信息。

### views

　　views表：记录了视图信息，需要有show view权限。

### triggers

　　triggers表：存储了触发器的信息，需要有super权限。

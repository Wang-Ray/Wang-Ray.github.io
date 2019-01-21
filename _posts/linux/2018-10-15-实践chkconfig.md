---
layout: post
title: "实践chkconfig"
date: 2018-10-15 13:53:00 +0800
categories: Linux
tags: linux chkconfig
---

chkconfig用于检查和设置系统的各种服务启动情况。

```shell
$ chkconfig --help
语  法：
chkconfig [--add][--del][--list][系统服务] 
或
chkconfig [--level <等级代号>][系统服务][on/off/reset]

补充说明：这是Red Hat公司遵循GPL规则所开发的程序，它可查询操作系统在每一个执行等级中会执行哪些系统服务，其中包括各类常驻服务。

参  数：
--add  增加所指定的系统服务，让chkconfig指令得以管理它，并同时在系统启动的叙述文件内增加相关数据。
--del  删除所指定的系统服务，不再由chkconfig指令管理，并同时在系统启动的叙述文件内删除相关数据。
--list 列举系统服务的启动信息
--level<等级代号>  指定读系统服务要在哪一个执行等级中开启或关毕
```

例如：

```shell
$ chkconfig --add memcached
$ chkconfig --del memcached
$ chkconfig --list memcached
$ chkconfig --level 2 memcached on
```




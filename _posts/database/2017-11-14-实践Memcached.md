---
layout: post
title: "实践Memcached"
date: 2017-11-14 14:05:00 +0800
categories: 数据库
tags: memcached nosql
---



## deepin

```shell
$ apt install memcached
$ sudo systemctl start memcached
```



1、根据系统情况，修改memcached文件中的用户名为相应应用用户（建议非root），参考：
......
$memcached_path -m $memcached_memory -l 0.0.0.0 -p 11211 -u itsapp -d -P $memcached_pid 
......

2、根据系统情况，修改memcached文件中的日志目录，比如：/itslog/

3、安装memcached（使用root用户）

```shell

$ cd libevent-2.1.8-stable/ && ./configure --prefix=/usr && make && make install

$ cd memcached-1.5.3 && ./configure --with-libevent=/usr/lib && make && make install

$ ln -s /usr/lib/libevent-2.1.so.6 /usr/lib64/libevent-2.1.so.6 && ldd /usr/local/bin/memcached

$ cp memcached /etc/init.d/memcached

$ cp memcached.conf /app/config/

$ chmod +x /etc/init.d/memcached

$ chkconfig --add memcached

$ chkconfig memcached on
```





1、根据系统情况，修改memcached文件中的用户名为相应应用用户（建议非root），参考：
......
$memcached_path -m $memcached_memory -l 0.0.0.0 -p 11211 -u itsapp -d -P $memcached_pid 
......

2、根据系统情况，修改memcached文件中的日志目录，比如：/itslog/

3、安装memcached（使用root用户）
cd libevent-2.1.8-stable/ && ./configure --prefix=/usr && make && make install
cd memcached-1.5.3 && ./configure --with-libevent=/usr/lib && make && make install
ln -s /usr/lib/libevent-2.1.so.6 /usr/lib64/libevent-2.1.so.6 && ldd /usr/local/bin/memcached
cp memcached /etc/init.d/memcached
chmod +x /etc/init.d/memcached
chkconfig --add memcached
chkconfig memcached on



/usr/local/bin/memcached -P /app/software/memcached.pid -d -u ctsapp -m 128 -l 0.0.0.0 -p 11211 -vvv > /app/log/memcached.log 2>&1
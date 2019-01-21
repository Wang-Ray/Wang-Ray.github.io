---
layout: post
title: "实践缓存中间件之twemproxy"
date: 2018-07-12 11:08:00 +0800
categories: 数据库
tags: database cache twemproxy
---

[twemproxy](https://github.com/twitter/twemproxy) (pronounced "two-em-proxy"), aka **nutcracker** is a fast and lightweight proxy for [memcached](http://www.memcached.org/) and [redis](http://redis.io/) protocol. It was built primarily to reduce the number of connections to the caching servers on the backend. This, together with protocol pipelining and sharding enables you to horizontally scale your distributed caching architecture.
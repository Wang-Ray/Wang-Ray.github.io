---
layout: post
title: "实践Sentinel之sentinel-transport"
date: 2019-11-22 11:08:00 +0800
categories: Java
tags: java Sentinel flow-control circuit-breaking
---

sentinel对外api

引入了 transport 模块后，可以通过以下的 HTTP API 来获取所有已加载的规则：

```
http://localhost:8719/getRules?type=<XXXX>
```

其中，`type=flow` 以 JSON 格式返回现有的限流规则，degrade 返回现有生效的降级规则列表，system 则返回系统保护规则。

获取所有热点规则：

```
http://localhost:8719/getParamRules
```



### WritableDataSourceRegistry

Writable data source registry for modifying rules via HTTP API.
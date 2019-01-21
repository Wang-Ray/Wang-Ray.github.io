---
layout: post
title: "实践CommonsHttpClient"
date: 2018-04-17 11:08:00 +0800
categories: Java
tags: java CommonsHttpClient HttpClient HttpComponents
---

[Commons HttpClient](http://hc.apache.org/httpclient-legacy/index.html)

### header

addRequestHeader(key,value)

### body

setRequestEntity(RequestEntity)

RequestEntity是对body的一种包装。

![RequestEntity](/images/CommonsHttpClient-RequestEntity.png)

setRequestBody(String)

setRequestBody(InputStream)

setParameter(name,value)：通过PostMethod#generateRequestEntity-》EncodingUtil#formUrlEncode转换为RequestEntity，字符串表示形式为`name1=value1&name2=value2`

### HTTP protocol parameters

```java
// 下面两种方法等价
postMethod.getParams().setParameter(HttpMethodParams.HTTP_CONTENT_CHARSET, "utf-8");
postMethod.getParams().setContentCharset("utf-8");
```

### connection manager

关闭相应的io对象即可，会自动回收连接到连接池。

![HttpConnectionManager](/images/CommonsHttpClient-HttpConnectionManager.png)

`HttpConnectionManager`有三个自带实现`SimpleHttpConnectionManager`、`MultiThreadedHttpConnectionManager`和

```java
MultiThreadedHttpConnectionManager multiThreadedHttpConnectionManager = new MultiThreadedHttpConnectionManager();
		multiThreadedHttpConnectionManager.getParams().setDefaultMaxConnectionsPerHost(maxHostConnections);
		multiThreadedHttpConnectionManager.getParams().setMaxTotalConnections(maxTotalConnections);
		client = new HttpClient(multiThreadedHttpConnectionManager);
```




---
layout: post
title: "实践HttpClient"
date: 2018-04-17 11:08:00 +0800
categories: Java
tags: java HttpClient HttpComponents
---

[HttpClient](http://hc.apache.org/)

### header

PostMethod.setHeader(key,value)

### body

PostMethod.setEntity(HttpEntity)

![HttpEntity](/images/HttpClient-HttpEntity.png)

### HTTP protocol parameters

PostMethod.setParams(HttpParams)

### connection manager

关闭相应的io对象即可，会自动回收连接到连接池。

@since 4.0

`ClientConnectionManager`有两个自带的实现`PoolingClientConnectionManager`和`BasicClientConnectionManager`

```java
ClientConnectionManager ccm = new PoolingClientConnectionManager();
((PoolingClientConnectionManager) ccm).setMaxTotal(10000);
((PoolingClientConnectionManager) ccm).setDefaultMaxPerRoute(300);
DefaultHttpClient httpclient = new DefaultHttpClient(ccm);
HttpConnectionParams.setConnectionTimeout(httpclient.getParams(),10*1000);
```

@since 4.3

`HttpClientConnectionManager`有两个自带的实现`PoolingHttpClientConnectionManager`和`BasicHttpClientConnectionManager`

```java
PoolingHttpClientConnectionManager poolingHttpClientConnectionManager = new PoolingHttpClientConnectionManager();
poolingHttpClientConnectionManager.setMaxTotal(10000);
poolingHttpClientConnectionManager.setDefaultMaxPerRoute(300);

RequestConfig requestConfig = RequestConfig.custom()
	.setConnectionRequestTimeout(CONNECTIONREQUEST_TIMEOUT)
	.setConnectTimeout(CONNECT_TIMEOUT)
	.setSocketTimeout(SOCKET_TIMEOUT).build();

httpClient = HttpClients.custom()
	.setConnectionManager(poolingHttpClientConnectionManager)
	.setDefaultRequestConfig(requestConfig)
	.build();
```



### Core

### Client

### AsyncClient


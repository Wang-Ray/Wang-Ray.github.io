---
layout: post
categories: Java
tags: Java Spring ServletContextInitializer
---



## 概述

`ServletContextInitializer`是 Servlet 容器初始化的时候，提供的初始化接口。实现包括初始化Servlet的ServletRegistrationBean、Filter的FilterRegistrationBean，Servlet的EventListener的ServletListenerRegistrationBean和DelegatingFilterProxy的DelegatingFilterProxyRegistrationBean等，以及根据需要自定义实现。

### 接口

```java
package org.springframework.boot.web.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;

import org.springframework.web.SpringServletContainerInitializer;
import org.springframework.web.WebApplicationInitializer;

/**
 * Interface used to configure a Servlet 3.0+ {@link ServletContext context}
 * programmatically. Unlike {@link WebApplicationInitializer}, classes that implement this
 * interface (and do not implement {@link WebApplicationInitializer}) will <b>not</b> be
 * detected by {@link SpringServletContainerInitializer} and hence will not be
 * automatically bootstrapped by the Servlet container.
 * <p>
 * This interface is primarily designed to allow {@link ServletContextInitializer}s to be
 * managed by Spring and not the Servlet container.
 * <p>
 * For configuration examples see {@link WebApplicationInitializer}.
 *
 * @author Phillip Webb
 * @since 1.4.0
 * @see WebApplicationInitializer
 */
@FunctionalInterface
public interface ServletContextInitializer {

	/**
	 * Configure the given {@link ServletContext} with any servlets, filters, listeners
	 * context-params and attributes necessary for initialization.
	 * @param servletContext the {@code ServletContext} to initialize
	 * @throws ServletException if any call against the given {@code ServletContext}
	 * throws a {@code ServletException}
	 */
	void onStartup(ServletContext servletContext) throws ServletException;

}
```

## 原理

### 

## 参考

[SpringBoot2 | SpingBoot FilterRegistrationBean 注册组件 | FilterChain 责任链源码分析（九）](https://blog.csdn.net/woshilijiuyi/article/details/85014183)
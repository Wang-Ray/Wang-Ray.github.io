---
layout: post
title: "实践Spring Boot 2之WebServer"
categories: Java
tags: java WebServer Tomcat Netty Jetty Undertow
---

Spring Boot 2

## WebServer

![WebServer](/images/WebServer.png)

```java
package org.springframework.boot.web.server;

/**
 * Simple interface that represents a fully configured web server (for example Tomcat,
 * Jetty, Netty). Allows the server to be {@link #start() started} and {@link #stop()
 * stopped}.
 *
 * @author Phillip Webb
 * @author Dave Syer
 * @since 2.0.0
 */
public interface WebServer {

	/**
	 * Starts the web server. Calling this method on an already started server has no
	 * effect.
	 * @throws WebServerException if the server cannot be started
	 */
	void start() throws WebServerException;

	/**
	 * Stops the web server. Calling this method on an already stopped server has no
	 * effect.
	 * @throws WebServerException if the server cannot be stopped
	 */
	void stop() throws WebServerException;

	/**
	 * Return the port this server is listening on.
	 * @return the port (or -1 if none)
	 */
	int getPort();

}

```

### ServletWebServerFactory

![ServletWebServerFactory](/images/ServletWebServerFactory.png)

Servlet WebServer

### ReactiveWebServerFactory

![ReactiveWebServerFactory](/images/ReactiveWebServerFactory.png)

反应式WebServer

## WebServerFactoryCustomizer

```java
@FunctionalInterface
public interface WebServerFactoryCustomizer<T extends WebServerFactory> {

	/**
	 * Customize the specified {@link WebServerFactory}.
	 * @param factory the web server factory to customize
	 */
	void customize(T factory);

}
```

WebServerFactoryCustomizerBeanPostProcessor实施执行WebServerFactoryCustomizer，支持N个实例。

```java
private Collection<WebServerFactoryCustomizer<?>> getCustomizers() {
		if (this.customizers == null) {
			// Look up does not include the parent context
			this.customizers = new ArrayList<>(getWebServerFactoryCustomizerBeans());
			this.customizers.sort(AnnotationAwareOrderComparator.INSTANCE);
			this.customizers = Collections.unmodifiableList(this.customizers);
		}
		return this.customizers;
	}

private void postProcessBeforeInitialization(WebServerFactory webServerFactory) {
		LambdaSafe
				.callbacks(WebServerFactoryCustomizer.class, getCustomizers(),
						webServerFactory)
				.withLogger(WebServerFactoryCustomizerBeanPostProcessor.class)
				.invoke((customizer) -> customizer.customize(webServerFactory));
	}
```



## NettyWebServer

### NettyReactiveWebServerFactory

创建NettyWebServer的工厂。

支持NettyServerCustomizer。

```java
/**
	 * Set {@link NettyServerCustomizer}s that should be applied to the Netty server
	 * builder. Calling this method will replace any existing customizers.
	 * @param serverCustomizers the customizers to set
	 */
	public void setServerCustomizers(
			Collection<? extends NettyServerCustomizer> serverCustomizers) {
		Assert.notNull(serverCustomizers, "ServerCustomizers must not be null");
		this.serverCustomizers = new ArrayList<>(serverCustomizers);
	}

	/**
	 * Add {@link NettyServerCustomizer}s that should applied while building the server.
	 * @param serverCustomizers the customizers to add
	 */
	public void addServerCustomizers(NettyServerCustomizer... serverCustomizers) {
		Assert.notNull(serverCustomizers, "ServerCustomizer must not be null");
		this.serverCustomizers.addAll(Arrays.asList(serverCustomizers));
	}
```

### NettyWebServerFactoryCustomizer

NettyWebServerFactory定制配置

### NettyServerCustomizer

NettyServer定制配置

```java
package org.springframework.boot.web.embedded.netty;

import java.util.function.Function;
import reactor.netty.http.server.HttpServer;
/**
 * Mapping function that can be used to customize a Reactor Netty server instance.
 *
 * @author Brian Clozel
 * @see NettyReactiveWebServerFactory
 * @since 2.1.0
 */
@FunctionalInterface
public interface NettyServerCustomizer extends Function<HttpServer, HttpServer> {

}
```

### HttpServer

reactor-netty基于Netty扩展的

## TomcatWebServer

### TomcatServletWebServerFactory

### TomcatReactiveWebServerFactory



## JettyWebServer

### JettyServletWebServerFactory

### JettyReactiveWebServerFactory



## UndertowWebServer

### UndertowReactiveWebServerFactory

## UndertowServletWebServer

### UndertowServletWebServerFactory

## ReactiveWebServerFactoryAutoConfiguration

## 参考


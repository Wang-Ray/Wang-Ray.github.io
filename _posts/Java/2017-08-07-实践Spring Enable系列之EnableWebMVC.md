---
layout: post
title: "实践Spring Enable系列之EnableWebMVC"
date: 2017-08-04 13:41:00 +0800
categories: Java
tags: spring web mvc spring-mvc
---

Adding this annotation to an `@Configuration` class imports the Spring MVC configuration from `WebMvcConfigurationSupport`, e.g.:

```java
 @Configuration
 @EnableWebMvc
 @ComponentScan(basePackageClasses = { MyConfiguration.class })
 public class MyWebConfiguration {

 }
 
```

Similar to support found in `<mvc:annotation-driven/>` namespace.

To customize the imported configuration, implement the interface `WebMvcConfigurer` or more likely extend the empty method base class `WebMvcConfigurerAdapter` and override individual methods, e.g.:

```java
 @Configuration
 @EnableWebMvc
 @ComponentScan(basePackageClasses = { MyConfiguration.class })
 public class MyConfiguration extends WebMvcConfigurerAdapter {

 	   @Override
 	   public void addFormatters(FormatterRegistry formatterRegistry) {
         formatterRegistry.addConverter(new MyConverter());
 	   }

 	   @Override
 	   public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
         converters.add(new MyHttpMessageConverter());
 	   }

     // More overridden methods ...
 }
```

If `WebMvcConfigurer` does not expose some advanced setting that needs to be configured, consider **removing** the `@EnableWebMvc` annotation and extending directly from `WebMvcConfigurationSupport`or `DelegatingWebMvcConfiguration`, e.g.:

```java
 @Configuration
 @ComponentScan(basePackageClasses = { MyConfiguration.class })
 public class MyConfiguration extends WebMvcConfigurationSupport {

 	   @Override
	   public void addFormatters(FormatterRegistry formatterRegistry) {
         formatterRegistry.addConverter(new MyConverter());
	   }

	   @Bean
	   public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
         // Create or delegate to "super" to create and
         // customize properties of RequestMappingHandlerAdapter
	   }
 }
```

### 原理分析

@EnableWebMvc实际导入了DelegatingWebMvcConfiguration

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

`````java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
...
}
`````

以下仅以interceptor为例分析原理，其他类似。

WebMvcConfigurationSupport中有如下拦截器相关的方法：

```java
protected final Object[] getInterceptors() {
	if (this.interceptors == null) {
		InterceptorRegistry registry = new InterceptorRegistry();
    	// 添加拦截器
		addInterceptors(registry);
		registry.addInterceptor(new ConversionServiceExposingInterceptor(mvcConversionService()));
		registry.addInterceptor(new ResourceUrlProviderExposingInterceptor(mvcResourceUrlProvider()));
		this.interceptors = registry.getInterceptors();
	}
	return this.interceptors.toArray();
}
// 由子类覆盖添加拦截器
protected void addInterceptors(InterceptorRegistry registry) {
}
```
DelegatingWebMvcConfiguration覆盖了addInterceptors(InterceptorRegistry)方法

```java
@Override
protected void addInterceptors(InterceptorRegistry registry) {
	this.configurers.addInterceptors(registry);
}
```
且

```java
private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

// 自动注入所有WebMvcConfigurer
@Autowired(required = false)
public void setConfigurers(List<WebMvcConfigurer> configurers) {
	if (configurers == null || configurers.isEmpty()) {
		return;
	}
	this.configurers.addWebMvcConfigurers(configurers);
}
```

WebMvcConfigurerComposite的addInterceptors实现如下：

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
	for (WebMvcConfigurer delegate : this.delegates) {
		delegate.addInterceptors(registry);
	}
}
```

可见添加拦截器最终代理给了`WebMvcConfigurer`，以及`WebMvcConfigurerAdapter`。

```java
public interface WebMvcConfigurer {
...
	void addInterceptors(InterceptorRegistry registry);
...
}
```

自定义拦截器通过InterceptorRegistry添加即可，参考如下：

```java
// 定义拦截器
@Bean
public DemoInterceptor demoInterceptor() {
	return new DemoInterceptor();
}

// 配置自定义拦截器
@Override
public void addInterceptors(InterceptorRegistry registry) {
	registry.addInterceptor(demoInterceptor());
}
```

Spring boot中参见[自动配置](/2017/08/22/实践Spring-Boot之EnableAutoConfiguration注解.html)之WebMvcAutoConfiguration。

### WebMvcConfigurationSupport

This is the main class providing the configuration behind the MVC Java config. It is typically imported by adding `@EnableWebMvc` to an application `@Configuration` class. An alternative more advanced option is to extend directly from this class and override methods as necessary remembering to add `@Configuration` to the subclass and `@Bean` to overridden `@Bean` methods. For more details see the Javadoc of `@EnableWebMvc`.

This class registers the following `HandlerMapping`s:

- `RequestMappingHandlerMapping` ordered at 0 for mapping requests to annotated controller methods.
- `HandlerMapping` ordered at 1 to map URL paths directly to view names.
- `BeanNameUrlHandlerMapping` ordered at 2 to map URL paths to controller bean names.
- `HandlerMapping` ordered at `Integer.MAX_VALUE-1` to serve static resource requests.
- `HandlerMapping` ordered at `Integer.MAX_VALUE` to forward requests to the default servlet.

Registers these `HandlerAdapter`s:

- `RequestMappingHandlerAdapter` for processing requests with annotated controller methods.
- `HttpRequestHandlerAdapter` for processing requests with `HttpRequestHandler`s.
- `SimpleControllerHandlerAdapter` for processing requests with interface-based `Controller`s.

Registers a `HandlerExceptionResolverComposite` with this chain of exception resolvers:

- `ExceptionHandlerExceptionResolver` for handling exceptions through @`ExceptionHandler`methods.
- `ResponseStatusExceptionResolver` for exceptions annotated with @`ResponseStatus`.
- `DefaultHandlerExceptionResolver` for resolving known Spring exception types

Both the `RequestMappingHandlerAdapter` and the `ExceptionHandlerExceptionResolver` are configured with default instances of the following by default:

- A `ContentNegotiationManager`
- A `DefaultFormattingConversionService`
- A `LocalValidatorFactoryBean` if a JSR-303 implementation is available on the classpath
- A range of `HttpMessageConverter`s depending on the 3rd party libraries available on the classpath.
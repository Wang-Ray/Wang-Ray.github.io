---
layout: post
title: "实践Spring Enable系列之EnableAsync"
date: 2017-08-05 13:41:00 +0800
categories: Java
tags: spring asynchronous thread-pool
---
## `EnableAsync`

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AsyncConfigurationSelector.class)
public @interface EnableAsync {
//...
}
```

Enables Spring's asynchronous method execution capability, similar to functionality found in Spring's `<task:*>` XML namespace. To be used on @`Configuration`classes as follows:

```java
 @Configuration
 @EnableAsync
 public class AppConfig {
     @Bean
     public MyAsyncBean asyncBean() {
         return new MyAsyncBean();
     }
 }
```

where MyAsyncBeanis a user-defined type with one or methods annotated with @Async (or any custom annotation specified by the annotation() attribute).

The `mode()` attribute controls how advice is applied; if the mode is `AdviceMode.PROXY` (the default), then the other attributes control the behavior of the proxying.

Note that if the mode is set to `AdviceMode.ASPECTJ`, then the `proxyTargetClass()` attribute is obsolete. Note also that in this case the `spring-aspects`module JAR must be present on the classpath.

By default, a `SimpleAsyncTaskExecutor` will be used to process async method invocations. To customize this behavior, implement `AsyncConfigurer` and provide your own `Executor` through the `getExecutor()` method.

```java
 @Configuration
 @EnableAsync
 public class AppConfig implements AsyncConfigurer {

     @Bean
     public MyAsyncBean asyncBean() {
         return new MyAsyncBean();
     }

     @Override
     public Executor getAsyncExecutor() {
         ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
         executor.setCorePoolSize(7);
         executor.setMaxPoolSize(42);
         executor.setQueueCapacity(11);
         executor.setThreadNamePrefix("MyExecutor-");
         executor.initialize();
         return executor;
     }
 }
```

For reference, the example above can be compared to the following Spring XML configuration:

```xml
 <beans>
     <task:annotation-driven executor="myExecutor"/>
     <task:executor id="myExecutor" pool-size="7-42" queue-capacity="11"/>
     <bean id="asyncBean" class="com.foo.MyAsyncBean"/>
 </beans>
```

the examples are equivalent save the setting of the thread name prefix of the Executor; this is because the the task:namespace executor element does not expose such an attribute. This demonstrates how the code-based approach allows for maximum configurability through direct access to actual componentry.

**Note**: In the above example the `ThreadPoolTaskExecutor` is not a fully managed Spring Bean. Add the `@Bean` annotation to the `getAsyncExecutor()`method if you want a fully managed bean. In such circumstances it is no longer necessary to manually call the `executor.initialize()` method as this will be invoked automatically when the bean is initialized.



### 解析

当mode是AdviceMode.PROXY时，proxyTargetClass决定是代理接口还是类，如果是接口，@Async必须是接口方法，否则可以是非接口方法。

```java
# 当proxyTargetClass配置为false时，接口方法@Async有效
@Resource
private HelloService helloService;

helloService.asyncDo(name);
```

非面向接口编程也是可以的

```java
@Resource
private HelloServiceImpl1 helloServiceImpl1;

helloServiceImpl1.asyncDo(name);
```



```java
# 当proxyTargetClass配置为true时，下面的非接口方法@Async有效
HelloServiceImpl helloService1 = (HelloServiceImpl)helloService;
helloService1.asyncDo1(name);
```



当mode配置为AdviceMode.PROXY时，本地方法调用@Async无效，必须`走代理`@Async才有效。

```java
# 当mode配置为AdviceMode.PROXY时，本地方法调用@Async无效
@Override
public void asyncDo(String name) {
	logger.info("HelloServiceImpl.asyncDo " + name);
	asyncDo1(name);
}

@Async
public void asyncDo1(String name) {
	logger.info("HelloServiceImpl.asyncDo1 " + name);
}
```



### 线程

默认线程池`SimpleAsyncTaskExecutor`，线程名SimpleAsyncTaskExecutor-1，SimpleAsyncTaskExecutor-2



## QA

### mode配置为AdviceMode.ASPECTJ，报错如下：

```
...
2023-04-07 13:28:44.048  WARN 15308 --- [           main] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanDefinitionStoreException: Failed to process import candidates for configuration class [wang.ray.dubbo.spring.boot.starter.sample.consumer.DubboRegistryZooKeeperConsumerBootstrap]; nested exception is java.io.FileNotFoundException: class path resource [org/springframework/scheduling/aspectj/AspectJAsyncConfiguration.class] cannot be opened because it does not exist
2023-04-07 13:28:44.282 ERROR 15308 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.BeanDefinitionStoreException: Failed to process import candidates for configuration class [wang.ray.dubbo.spring.boot.starter.sample.consumer.DubboRegistryZooKeeperConsumerBootstrap]; nested exception is java.io.FileNotFoundException: class path resource [org/springframework/scheduling/aspectj/AspectJAsyncConfiguration.class] cannot be opened because it does not exist
	at org.springframework.context.annotation.ConfigurationClassParser.processImports(ConfigurationClassParser.java:609) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassParser.doProcessConfigurationClass(ConfigurationClassParser.java:310) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassParser.processConfigurationClass(ConfigurationClassParser.java:249) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassParser.parse(ConfigurationClassParser.java:206) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassParser.parse(ConfigurationClassParser.java:174) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.processConfigBeanDefinitions(ConfigurationClassPostProcessor.java:319) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry(ConfigurationClassPostProcessor.java:236) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanDefinitionRegistryPostProcessors(PostProcessorRegistrationDelegate.java:280) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(PostProcessorRegistrationDelegate.java:96) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.invokeBeanFactoryPostProcessors(AbstractApplicationContext.java:707) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:533) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:143) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:755) [spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:747) [spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:402) [spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:312) [spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1247) [spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1236) [spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
	at wang.ray.dubbo.spring.boot.starter.sample.consumer.DubboRegistryZooKeeperConsumerBootstrap.main(DubboRegistryZooKeeperConsumerBootstrap.java:49) [classes/:na]
Caused by: java.io.FileNotFoundException: class path resource [org/springframework/scheduling/aspectj/AspectJAsyncConfiguration.class] cannot be opened because it does not exist
	at org.springframework.core.io.ClassPathResource.getInputStream(ClassPathResource.java:180) ~[spring-core-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.core.type.classreading.SimpleMetadataReader.getClassReader(SimpleMetadataReader.java:55) ~[spring-core-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.core.type.classreading.SimpleMetadataReader.<init>(SimpleMetadataReader.java:49) ~[spring-core-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.core.type.classreading.SimpleMetadataReaderFactory.getMetadataReader(SimpleMetadataReaderFactory.java:103) ~[spring-core-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.boot.type.classreading.ConcurrentReferenceCachingMetadataReaderFactory.createMetadataReader(ConcurrentReferenceCachingMetadataReaderFactory.java:86) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
	at org.springframework.boot.type.classreading.ConcurrentReferenceCachingMetadataReaderFactory.getMetadataReader(ConcurrentReferenceCachingMetadataReaderFactory.java:73) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
	at org.springframework.core.type.classreading.SimpleMetadataReaderFactory.getMetadataReader(SimpleMetadataReaderFactory.java:81) ~[spring-core-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassParser.asSourceClass(ConfigurationClassParser.java:695) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassParser.asSourceClasses(ConfigurationClassParser.java:674) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassParser.processImports(ConfigurationClassParser.java:581) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	... 18 common frames omitted


Process finished with exit code 1
```

新增依赖解决：

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-aspects</artifactId>
</dependency>
```



### 当proxyTargetClass配置为false时，如下代码：

```java
HelloServiceImpl helloService1 = (HelloServiceImpl)helloService;
helloService1.asyncDo1(name);
```

运行时报错：

```
2023-04-07 13:43:31.145 ERROR 15744 --- [nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.ClassCastException: com.sun.proxy.$Proxy72 cannot be cast to wang.ray.dubbo.spring.boot.starter.sample.consumer.service.impl.HelloServiceImpl] with root cause

java.lang.ClassCastException: com.sun.proxy.$Proxy72 cannot be cast to wang.ray.dubbo.spring.boot.starter.sample.consumer.service.impl.HelloServiceImpl
	at wang.ray.dubbo.spring.boot.starter.sample.consumer.controller.HelloController.asyncHello(HelloController.java:56) ~[classes/:na]
	at wang.ray.dubbo.spring.boot.starter.sample.consumer.controller.HelloController$$FastClassBySpringCGLIB$$215ecd69.invoke(<generated>) ~[classes/:na]
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:687) ~[spring-aop-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at wang.ray.dubbo.spring.boot.starter.sample.consumer.controller.HelloController$$EnhancerBySpringCGLIB$$f55c245a.asyncHello(<generated>) ~[classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_152]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_152]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_152]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_152]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190) ~[spring-web-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:138) ~[spring-web-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:105) ~[spring-webmvc-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:878) ~[spring-webmvc-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:792) ~[spring-webmvc-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1040) ~[spring-webmvc-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:943) ~[spring-webmvc-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006) ~[spring-webmvc-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898) ~[spring-webmvc-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:626) ~[tomcat-embed-core-9.0.46.jar:4.0.FR]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883) ~[spring-webmvc-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:733) ~[tomcat-embed-core-9.0.46.jar:4.0.FR]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:227) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) ~[tomcat-embed-websocket-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100) ~[spring-web-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) ~[spring-web-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93) ~[spring-web-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) ~[spring-web-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201) ~[spring-web-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) ~[spring-web-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:542) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:143) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:357) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:374) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:893) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1707) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_152]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_152]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_152]

```


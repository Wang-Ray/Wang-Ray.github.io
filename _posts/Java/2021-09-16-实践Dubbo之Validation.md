---
layout: post
title: "实践Dubbo之Validation"
categories: Java
tags: Java Dubbo Validation
---

## 关键字

### ValidationException

### ConstraintViolationException



## 样例

dubbo 2.7.13

```xml
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
	<version>2.0.1.Final</version>
</dependency>
<dependency>
	<groupId>org.hibernate.validator</groupId>
	<artifactId>hibernate-validator</artifactId>
	<version>6.1.5.Final</version>
</dependency>
<dependency>
	<groupId>org.glassfish</groupId>
	<artifactId>javax.el</artifactId>
	<version>3.0.1-b12</version>
</dependency>
```

配置`validation`属性为`true`

缺少`javax.el`依赖则报错如下：
```
javax.validation.ValidationException: HV000183: Unable to initialize 'javax.el.ExpressionFactory'. Check that you have the EL dependencies on the classpath, or use ParameterMessageInterpolator instead
	at org.apache.dubbo.validation.filter.ValidationFilter.invoke(ValidationFilter.java:96) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:91) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.dubbo.filter.FutureFilter.invoke(FutureFilter.java:52) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.filter.ConsumerContextFilter.invoke(ConsumerContextFilter.java:69) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.proxy.InvokerInvocationHandler.invoke(InvokerInvocationHandler.java:96) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.common.bytecode.proxy0.getBook(proxy0.java) ~[dubbo-2.7.13.jar:2.7.13]
	at wang.ray.dubbo.spring.boot.starter.sample.provider.test.SampleRemoteServiceTests.testGetBook(SampleRemoteServiceTests.java:46) ~[test-classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_152]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_152]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_152]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_152]
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:59) [junit-4.13.jar:4.13]
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12) [junit-4.13.jar:4.13]
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:56) [junit-4.13.jar:4.13]
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17) [junit-4.13.jar:4.13]
	at org.springframework.test.context.junit4.statements.RunBeforeTestExecutionCallbacks.evaluate(RunBeforeTestExecutionCallbacks.java:74) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.RunAfterTestExecutionCallbacks.evaluate(RunAfterTestExecutionCallbacks.java:84) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.RunBeforeTestMethodCallbacks.evaluate(RunBeforeTestMethodCallbacks.java:75) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.RunAfterTestMethodCallbacks.evaluate(RunAfterTestMethodCallbacks.java:86) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.SpringRepeat.evaluate(SpringRepeat.java:84) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:366) [junit-4.13.jar:4.13]
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:251) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:97) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.junit.runners.ParentRunner$4.run(ParentRunner.java:331) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:79) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:329) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner.access$100(ParentRunner.java:66) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:293) [junit-4.13.jar:4.13]
	at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:70) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.junit.runners.ParentRunner$3.evaluate(ParentRunner.java:306) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner.run(ParentRunner.java:413) [junit-4.13.jar:4.13]
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:190) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137) [junit-4.13.jar:4.13]
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69) [junit-rt.jar:na]
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33) [junit-rt.jar:na]
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:235) [junit-rt.jar:na]
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:54) [junit-rt.jar:na]
```

非泛化调用参数校验不通过异常堆栈（单元测试）：
```
javax.validation.ValidationException: Failed to validate service: wang.ray.dubbo.sample.SampleRemoteService, method: getBook, cause: [ConstraintViolationImpl{interpolatedMessage='id不能大于9999', propertyPath=id, rootBeanClass=class wang.ray.dubbo.sample.BookRequest, messageTemplate='id不能大于9999'}, ConstraintViolationImpl{interpolatedMessage='name至少6个字符长度', propertyPath=name, rootBeanClass=class wang.ray.dubbo.sample.BookRequest, messageTemplate='name至少6个字符长度'}]
	at org.apache.dubbo.validation.filter.ValidationFilter.invoke(ValidationFilter.java:96) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:91) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.dubbo.filter.FutureFilter.invoke(FutureFilter.java:52) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.filter.ConsumerContextFilter.invoke(ConsumerContextFilter.java:69) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.proxy.InvokerInvocationHandler.invoke(InvokerInvocationHandler.java:96) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.common.bytecode.proxy0.getBook(proxy0.java) ~[dubbo-2.7.13.jar:2.7.13]
	at wang.ray.dubbo.spring.boot.starter.sample.provider.test.SampleRemoteServiceTests.testGetBook(SampleRemoteServiceTests.java:46) ~[test-classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_152]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_152]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_152]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_152]
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:59) [junit-4.13.jar:4.13]
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12) [junit-4.13.jar:4.13]
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:56) [junit-4.13.jar:4.13]
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17) [junit-4.13.jar:4.13]
	at org.springframework.test.context.junit4.statements.RunBeforeTestExecutionCallbacks.evaluate(RunBeforeTestExecutionCallbacks.java:74) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.RunAfterTestExecutionCallbacks.evaluate(RunAfterTestExecutionCallbacks.java:84) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.RunBeforeTestMethodCallbacks.evaluate(RunBeforeTestMethodCallbacks.java:75) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.RunAfterTestMethodCallbacks.evaluate(RunAfterTestMethodCallbacks.java:86) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.SpringRepeat.evaluate(SpringRepeat.java:84) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:366) [junit-4.13.jar:4.13]
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:251) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:97) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.junit.runners.ParentRunner$4.run(ParentRunner.java:331) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:79) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:329) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner.access$100(ParentRunner.java:66) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:293) [junit-4.13.jar:4.13]
	at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:70) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.junit.runners.ParentRunner$3.evaluate(ParentRunner.java:306) [junit-4.13.jar:4.13]
	at org.junit.runners.ParentRunner.run(ParentRunner.java:413) [junit-4.13.jar:4.13]
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:190) [spring-test-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137) [junit-4.13.jar:4.13]
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69) [junit-rt.jar:na]
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33) [junit-rt.jar:na]
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:235) [junit-rt.jar:na]
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:54) [junit-rt.jar:na]
```
非泛化调用参数校验不通过异常堆栈（consumer）：

```
javax.validation.ValidationException: Failed to validate service: wang.ray.dubbo.sample.SampleRemoteService, method: getBook, cause: [ConstraintViolationImpl{interpolatedMessage='id不能小于1000', propertyPath=id, rootBeanClass=class wang.ray.dubbo.sample.BookRequest, messageTemplate='id不能小于1000'}, ConstraintViolationImpl{interpolatedMessage='name至少6个字符长度', propertyPath=name, rootBeanClass=class wang.ray.dubbo.sample.BookRequest, messageTemplate='name至少6个字符长度'}]
        at org.apache.dubbo.validation.filter.ValidationFilter.invoke(ValidationFilter.java:96) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.dubbo.filter.TraceFilter.invoke(TraceFilter.java:77) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.filter.TimeoutFilter.invoke(TimeoutFilter.java:46) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:91) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.filter.ExceptionFilter.invoke(ExceptionFilter.java:52) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.filter.GenericFilter.invoke(GenericFilter.java:192) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.filter.ClassLoaderFilter.invoke(ClassLoaderFilter.java:38) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.filter.EchoFilter.invoke(EchoFilter.java:41) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.filter.ContextFilter.invoke(ContextFilter.java:129) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol$1.reply(DubboProtocol.java:148) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.handleRequest(HeaderExchangeHandler.java:100) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.received(HeaderExchangeHandler.java:175) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.remoting.transport.DecodeHandler.received(DecodeHandler.java:51) ~[dubbo-2.7.13.jar:2.7.13]
        at org.apache.dubbo.remoting.transport.dispatcher.ChannelEventRunnable.run(ChannelEventRunnable.java:57) ~[dubbo-2.7.13.jar:2.7.13]
        at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128) ~[na:na]
        at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628) ~[na:na]
        at org.apache.dubbo.common.threadlocal.InternalRunnable.run(InternalRunnable.java:41) ~[dubbo-2.7.13.jar:2.7.13]
        at java.base/java.lang.Thread.run(Thread.java:834) ~[na:na]
```

泛化调用时参数校验不通过异常堆栈：
有个疑问：为啥是`com.alibaba.dubbo.rpc.service.GenericException`，而不是`org.apache.dubbo.rpc.service.GenericException`

```
com.alibaba.dubbo.rpc.service.GenericException: javax.validation.ValidationException: Failed to validate service: wang.ray.dubbo.sample.SampleRemoteService, method: getBook, cause: [ConstraintViolationImpl{interpolatedMessage='name至少6个字符长度', propertyPath=name, rootBeanClass=class wang.ray.dubbo.sample.BookRequest, messageTemplate='name至少6个字符长度'}, ConstraintViolationImpl{interpolatedMessage='id不能小于1000', propertyPath=id, rootBeanClass=class wang.ray.dubbo.sample.BookRequest, messageTemplate='id不能小于1000'}]
javax.validation.ValidationException: Failed to validate service: wang.ray.dubbo.sample.SampleRemoteService, method: getBook, cause: [ConstraintViolationImpl{interpolatedMessage='name至少6个字符长度', propertyPath=name, rootBeanClass=class wang.ray.dubbo.sample.BookRequest, messageTemplate='name至少6个字符长度'}, ConstraintViolationImpl{interpolatedMessage='id不能小于1000', propertyPath=id, rootBeanClass=class wang.ray.dubbo.sample.BookRequest, messageTemplate='id不能小于1000'}]
	at org.apache.dubbo.validation.filter.ValidationFilter.invoke(ValidationFilter.java:96)
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61)
	at org.apache.dubbo.rpc.protocol.dubbo.filter.TraceFilter.invoke(TraceFilter.java:77)
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61)
	at org.apache.dubbo.rpc.filter.TimeoutFilter.invoke(TimeoutFilter.java:46)
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61)
	at org.apache.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:91)
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61)
	at org.apache.dubbo.rpc.filter.ExceptionFilter.invoke(ExceptionFilter.java:52)
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61)
	at org.apache.dubbo.rpc.filter.GenericFilter.invoke(GenericFilter.java:187)
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61)
	at org.apache.dubbo.rpc.filter.ClassLoaderFilter.invoke(ClassLoaderFilter.java:38)
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61)
	at org.apache.dubbo.rpc.filter.EchoFilter.invoke(EchoFilter.java:41)
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61)
	at org.apache.dubbo.rpc.filter.ContextFilter.invoke(ContextFilter.java:129)
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61)
	at org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol$1.reply(DubboProtocol.java:148)
	at org.apache.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.handleRequest(HeaderExchangeHandler.java:100)
	at org.apache.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.received(HeaderExchangeHandler.java:175)
	at org.apache.dubbo.remoting.transport.DecodeHandler.received(DecodeHandler.java:51)
	at org.apache.dubbo.remoting.transport.dispatcher.ChannelEventRunnable.run(ChannelEventRunnable.java:57)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at org.apache.dubbo.common.threadlocal.InternalRunnable.run(InternalRunnable.java:41)
	at java.lang.Thread.run(Thread.java:748)

	at org.apache.dubbo.rpc.filter.GenericFilter.onResponse(GenericFilter.java:229) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.lambda$invoke$0(FilterNode.java:99) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.AsyncRpcResult.lambda$whenCompleteWithContext$0(AsyncRpcResult.java:197) ~[dubbo-2.7.13.jar:2.7.13]
	at java.util.concurrent.CompletableFuture.uniWhenComplete(CompletableFuture.java:760) ~[na:1.8.0_152]
	at java.util.concurrent.CompletableFuture.uniWhenCompleteStage(CompletableFuture.java:778) ~[na:1.8.0_152]
	at java.util.concurrent.CompletableFuture.whenComplete(CompletableFuture.java:2140) ~[na:1.8.0_152]
	at org.apache.dubbo.rpc.AsyncRpcResult.whenCompleteWithContext(AsyncRpcResult.java:195) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:81) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.filter.ClassLoaderFilter.invoke(ClassLoaderFilter.java:38) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.filter.EchoFilter.invoke(EchoFilter.java:41) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.filter.ContextFilter.invoke(ContextFilter.java:129) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.FilterNode.invoke(FilterNode.java:61) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol$1.reply(DubboProtocol.java:148) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.handleRequest(HeaderExchangeHandler.java:100) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.received(HeaderExchangeHandler.java:175) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.remoting.transport.DecodeHandler.received(DecodeHandler.java:51) ~[dubbo-2.7.13.jar:2.7.13]
	at org.apache.dubbo.remoting.transport.dispatcher.ChannelEventRunnable.run(ChannelEventRunnable.java:57) ~[dubbo-2.7.13.jar:2.7.13]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) ~[na:1.8.0_152]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) ~[na:1.8.0_152]
	at org.apache.dubbo.common.threadlocal.InternalRunnable.run(InternalRunnable.java:41) ~[dubbo-2.7.13.jar:2.7.13]
	at java.lang.Thread.run(Thread.java:748) ~[na:1.8.0_152]
```

Dubbo客户端会进行校验





```java
(t instanceof ValidationException) {
            return ApiResultDTO.fail(ExceptionCode.VALIDATION_ERROR, t.getMessage().substring(t.getMessage().indexOf("interpolatedMessage='") + 21, t.getMessage().indexOf("',")));
        } 
```



### ValidationFilter.java

把ConstraintViolationException异常丢掉了，因为[avoid of serialization exception for javax.validation.ConstraintViolationException](https://github.com/apache/dubbo/pull/5672)，其实是因为`ConstraintViolation`没有支持序列化

```java
/**
     * Perform the validation of before invoking the actual method based on <b>validation</b> attribute value.
     * @param invoker    service
     * @param invocation invocation.
     * @return Method invocation result
     * @throws RpcException Throws RpcException if  validation failed or any other runtime exception occurred.
     */
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        if (validation != null && !invocation.getMethodName().startsWith("$")
                && ConfigUtils.isNotEmpty(invoker.getUrl().getMethodParameter(invocation.getMethodName(), VALIDATION_KEY))) {
            try {
                Validator validator = validation.getValidator(invoker.getUrl());
                if (validator != null) {
                    validator.validate(invocation.getMethodName(), invocation.getParameterTypes(), invocation.getArguments());
                }
            } catch (RpcException e) {
                throw e;
            } catch (ValidationException e) {
                // only use exception's message to avoid potential serialization issue
                return AsyncRpcResult.newDefaultAsyncResult(new ValidationException(e.getMessage()), invocation);
            } catch (Throwable t) {
                return AsyncRpcResult.newDefaultAsyncResult(t, invocation);
            }
        }
        return invoker.invoke(invocation);
    }
```

老版本2.7.5

```java
 /**
     * Perform the validation of before invoking the actual method based on <b>validation</b> attribute value.
     * @param invoker    service
     * @param invocation invocation.
     * @return Method invocation result
     * @throws RpcException Throws RpcException if  validation failed or any other runtime exception occurred.
     */
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        if (validation != null && !invocation.getMethodName().startsWith("$")
                && ConfigUtils.isNotEmpty(invoker.getUrl().getMethodParameter(invocation.getMethodName(), VALIDATION_KEY))) {
            try {
                Validator validator = validation.getValidator(invoker.getUrl());
                if (validator != null) {
                    validator.validate(invocation.getMethodName(), invocation.getParameterTypes(), invocation.getArguments());
                }
            } catch (RpcException e) {
                throw e;
            } catch (Throwable t) {
                return AsyncRpcResult.newDefaultAsyncResult(t, invocation);
            }
        }
        return invoker.invoke(invocation);
    }
```


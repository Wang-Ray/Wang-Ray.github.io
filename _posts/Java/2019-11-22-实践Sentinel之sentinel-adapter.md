---
layout: post
title: "实践Sentinel之sentinel-adapter"
date: 2019-11-22 11:08:00 +0800
categories: Java
tags: java Sentinel flow-control circuit-breaking
---

为主流框架提供适配，按照适配中的说明配置，Sentinel 就会默认定义提供的服务，方法等为资源。



```
com.alibaba.csp.sentinel.slots.block.SentinelRpcException: com.alibaba.csp.sentinel.slots.block.flow.FlowException
Caused by: com.alibaba.csp.sentinel.slots.block.flow.FlowException
```



```
org.apache.dubbo.rpc.RpcException: Failed to invoke remote method: sayHello, provider: dubbo://127.0.0.1:25758/com.alibaba.csp.sentinel.demo.apache.dubbo.FooService?application=demo-consumer&default.generic=false&default.lazy=false&default.sticky=false&dubbo=2.0.2&generic=false&interface=com.alibaba.csp.sentinel.demo.apache.dubbo.FooService&lazy=false&methods=doAnother,sayHello&pid=11453&register.ip=127.0.1.1&side=consumer&sticky=false&timeout=3000&timestamp=1574392753808, cause: com.alibaba.csp.sentinel.slots.block.SentinelRpcException: com.alibaba.csp.sentinel.slots.block.flow.FlowException
com.alibaba.csp.sentinel.slots.block.SentinelRpcException: com.alibaba.csp.sentinel.slots.block.flow.FlowException
Caused by: com.alibaba.csp.sentinel.slots.block.flow.FlowException

	at org.apache.dubbo.rpc.protocol.dubbo.DubboInvoker.doInvoke(DubboInvoker.java:113)
	at org.apache.dubbo.rpc.protocol.AbstractInvoker.invoke(AbstractInvoker.java:157)
	at com.alibaba.csp.sentinel.adapter.dubbo.SentinelDubboConsumerFilter.invoke(SentinelDubboConsumerFilter.java:62)
	at org.apache.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:73)
	at org.apache.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:88)
	at org.apache.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:73)
	at com.alibaba.csp.sentinel.adapter.dubbo.DubboAppContextFilter.invoke(DubboAppContextFilter.java:41)
	at org.apache.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:73)
	at org.apache.dubbo.rpc.protocol.dubbo.filter.FutureFilter.invoke(FutureFilter.java:49)
	at org.apache.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:73)
	at org.apache.dubbo.rpc.filter.ConsumerContextFilter.invoke(ConsumerContextFilter.java:54)
	at org.apache.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:73)
	at org.apache.dubbo.rpc.listener.ListenerInvokerWrapper.invoke(ListenerInvokerWrapper.java:78)
	at org.apache.dubbo.rpc.proxy.InvokerInvocationHandler.invoke(InvokerInvocationHandler.java:57)
	at org.apache.dubbo.common.bytecode.proxy0.sayHello(proxy0.java)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.dubbo.config.spring.beans.factory.annotation.ReferenceAnnotationBeanPostProcessor$ReferenceBeanInvocationHandler.invoke(ReferenceAnnotationBeanPostProcessor.java:165)
	at com.sun.proxy.$Proxy20.sayHello(Unknown Source)
	at com.alibaba.csp.sentinel.demo.apache.dubbo.consumer.FooServiceConsumer.sayHello(FooServiceConsumer.java:31)
	at com.alibaba.csp.sentinel.demo.apache.dubbo.FooConsumerBootstrap.main(FooConsumerBootstrap.java:49)
Caused by: org.apache.dubbo.remoting.RemotingException: com.alibaba.csp.sentinel.slots.block.SentinelRpcException: com.alibaba.csp.sentinel.slots.block.flow.FlowException
com.alibaba.csp.sentinel.slots.block.SentinelRpcException: com.alibaba.csp.sentinel.slots.block.flow.FlowException
Caused by: com.alibaba.csp.sentinel.slots.block.flow.FlowException

	at org.apache.dubbo.remoting.exchange.support.DefaultFuture.returnFromResponse(DefaultFuture.java:297)
	at org.apache.dubbo.remoting.exchange.support.DefaultFuture.get(DefaultFuture.java:191)
	at org.apache.dubbo.remoting.exchange.support.DefaultFuture.get(DefaultFuture.java:164)
	at org.apache.dubbo.rpc.protocol.dubbo.DubboInvoker.doInvoke(DubboInvoker.java:108)
	... 22 more
```


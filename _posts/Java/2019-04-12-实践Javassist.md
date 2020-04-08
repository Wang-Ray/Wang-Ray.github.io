---
layout: post
title: "实践Javassist"
date: 2019-04-12 11:08:00 +0800
categories: Java
tags: Java Javassist 动态编程
---

[Javassist](http://www.javassist.org/) Java bytecode engineering toolkit since 1999

Javassist (Java Programming Assistant) makes Java bytecode manipulation simple. It is a class library for editing bytecodes in Java; it enables Java programs to define a new class at runtime and to modify a class file when the JVM loads it. Unlike other similar bytecode editors, Javassist provides two levels of API: source level and bytecode level. If the users use the source-level API, they can edit a class file without knowledge of the specifications of the Java bytecode. The whole API is designed with only the vocabulary of the Java language. You can even specify inserted bytecode in the form of source text; Javassist compiles it on the fly. On the other hand, the bytecode-level API allows the users to directly edit a class file as other editors.



## ProxyFactory

既可以代理类，也可以代理接口

## MethodHandler

```java
package javassist.util.proxy;

import java.lang.reflect.Method;

/**
 * The interface implemented by the invocation handler of a proxy
 * instance.
 *
 * @see Proxy#setHandler(MethodHandler)
 */
public interface MethodHandler {
    /**
     * Is called when a method is invoked on a proxy instance associated
     * with this handler.  This method must process that method invocation.
     *
     * @param self          the proxy instance.
     * @param thisMethod    the overridden method declared in the super
     *                      class or interface.
     * @param proceed       the forwarder method for invoking the overridden 
     *                      method.  It is null if the overridden method is
     *                      abstract or declared in the interface.
     * @param args          an array of objects containing the values of
     *                      the arguments passed in the method invocation
     *                      on the proxy instance.  If a parameter type is
     *                      a primitive type, the type of the array element
     *                      is a wrapper class.
     * @return              the resulting value of the method invocation.
     *
     * @throws Throwable    if the method invocation fails.
     */
    Object invoke(Object self, Method thisMethod, Method proceed,
                  Object[] args) throws Throwable;
}
```

### invoke

Object self：生成的代理类的实例

Method thisMethod：被代理类或接口的方法

Method proceed：生成的代理类的方法，可能为null（抽象方法或代理接口时）

Object[] args：方法调用参数

```java
Object result = thisMethod.invoke(target, args);
```



```java
Object result = proceed.invoke(self, args);
```

## 样例



## 参考

[秒懂Java动态编程（Javassist研究）](https://blog.csdn.net/ShuSheng0007/article/details/81269295)

[Javassist的动态代理实现。](https://blog.csdn.net/mingxin95/article/details/51810499)


---
layout: post
title: "实践动态代理（Dynamic Proxy）之CGLIB"
categories: Java
tags: Java Proxy Dynamic-Proxy CGLIB
---

[CGLIB](https://github.com/cglib/cglib) Byte Code Generation Library is high level API to generate and transform Java byte code. It is used by AOP, testing, data access frameworks to generate dynamic proxy objects and intercept field access.

## 样例

```
public class Frank {
   public void submit(String proof) {
       System.out.println(String.format("老板欠薪跑路，证据如下：%s",proof));
   }
   public void defend() {
       System.out.println(String.format("铁证如山，%s还Frank血汗钱","马旭"));
   }
}
```



```
public class cgLibDynProxyLawyer implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] params, MethodProxy methodProxy) throws Throwable {
        if (method.getName().equals("submit"))
            System.out.println("案件提交成功,证据如下："+ Arrays.asList(params));
        Object result = methodProxy.invokeSuper(o, params);
        return result;
    }
}
```



```
public class ProxyFactory {
    public static Object getGcLibDynProxy(Object target){
        Enhancer enhancer=new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(new cgLibDynProxyLawyer());
        Object targetProxy= enhancer.create();
        return targetProxy;
    }
}
```



```
public static void main(String[] args) {
        Frank cProxy= (Frank) ProxyFactory.getGcLibDynProxy(new Frank());
        cProxy.submit("工资流水在此");
        cProxy.defend();
    }
```

## 原理



[Java Proxy 和 CGLIB 动态代理原理](http://www.importnew.com/27772.html)
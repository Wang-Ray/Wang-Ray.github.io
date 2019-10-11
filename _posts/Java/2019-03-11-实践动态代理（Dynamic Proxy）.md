---
layout: post
title: "实践动态代理（Dynamic Proxy）"
categories: Java
tags: Java Proxy Dynamic-Proxy
---

## 介绍

在java的动态代理机制中，有两个重要的类或接口，一个是InvocationHandler接口、另一个则是 Proxy类。InvocationHandler接口是给动态代理类实现的，负责处理被代理对象的操作的，而Proxy是用来创建动态代理类实例对象的，因为只有得到了这个对象我们才能调用那些需要代理的方法。

## 样例

```
public interface ILawSuit {
    void submit(String proof);//提起诉讼
    void defend();//法庭辩护
}
```



```
public class CuiHuaNiu implements ILawSuit {
    @Override
    public void submit(String proof) {
        System.out.println(String.format("老板欠薪跑路，证据如下：%s",proof));
    }
    @Override
    public void defend() {
        System.out.println(String.format("铁证如山，%s还牛翠花血汗钱","马旭"));
    }
}
```



```
public class DynProxyLawyer implements InvocationHandler {
    private Object target;//被代理的对象
    public DynProxyLawyer(Object obj){
        this.target=obj;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	    System.out.println("案件进展："+method.getName());
        Object result=method.invoke(target,args);
        return result;
    }
}
```



```
public class ProxyFactory {
	...

    public static Object getDynProxy(Object target) {
        InvocationHandler handler = new DynProxyLawyer(target);
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), handler);
    }
}
```



```
 public static void main(String[] args) {
        ILawSuit proxy= (ILawSuit) ProxyFactory.getDynProxy(new CuiHuaNiu());
        proxy.submit("工资流水在此");
        proxy.defend();
    }
```

## 原理





[秒懂Java代理与动态代理模式](https://blog.csdn.net/ShuSheng0007/article/details/80864854)

[Java的动态代理(dynamic proxy)](https://www.cnblogs.com/techyc/p/3455950.html)

[Java Proxy 和 CGLIB 动态代理原理](http://www.importnew.com/27772.html)
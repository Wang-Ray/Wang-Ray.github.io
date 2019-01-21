---
layout: post
title: "实践Mock框架之Mockito"
date: 2018-08-22 11:22:00 +0800
categories: 测试
tags: test Mock Mockito
---

[Mockito](https://site.mockito.org/), Automatic Test Suite Generation for Java

## 模拟对象

### Mockito.mock()

```java
// mock interfaces
List list = mock(List.class);
// mock concrete classes
ArrayList arrayList = mock(ArrayList.class);
```

### @Mock

```java
// 单纯的mock，不支持依赖注入
@Mock
private IBookDao bookDao;
```

### @InjectMocks

```java
// 支持依赖注入
@InjectMocks
private IHelloService helloService;
```

### @MockBean

spring-boot-test提供的

```java
// 支持依赖注入
@MockBean
private IHelloService helloService;
```

## stubbing

打桩，设定方法调用和返回。

```java
when(helloService.hello("Ray")).thenReturn("Hello, Ray");
```

### when

### 参数

ArgumentMatchers

```java
when(helloService.hello(anyString())).thenReturn("Hello, Ray");
when(helloService.hello(any(String.class))).thenReturn("Hello, Ray");
// 精确定义调用参数，如果实际调用时参数不符则不通过
when(helloService.hello(eq("Ray"))).thenReturn("Hello, Ray");
```

### 返回

OngoingStubbing

#### thenReturn()

#### thenThrow()

## 使用



## verify

VerificationMode

### 次数

Times

验证`方法（包括参数）`实际被调用情况，比如至少一次、精确次数或没有被调用等。

```java
verify(mock).someMethod("was called only once");

verify(mock, only()).someMethod("was called only once");

verify(mock, times(5)).someMethod("was called five times");

verify(mock, never()).someMethod("was never called");

verify(mock, atLeastOnce()).someMethod("was called at least once");

verify(mock, atLeast(2)).someMethod("was called at least twice");

verify(mock, atMost(3)).someMethod("was called at most 3 times");
```



```
verifyNoMoreInvocations(mock);
verifyZeroInteractions(mock);
```



### 超时

一般用于在多线程场景。

#### timeout

Timeout

验证是否在接下来指定时间内达到了指定的调用次数

```java
//passes when someMethod() is called no later than within 100 ms
         //exits immediately when verification is satisfied (e.g. may not wait full 100 ms)
         verify(mock, timeout(100)).someMethod();
         //above is an alias to:
         verify(mock, timeout(100).times(1)).someMethod();
      
         //passes as soon as someMethod() has been called 2 times under 100 ms
         verify(mock, timeout(100).times(2)).someMethod();
      
         //equivalent: this also passes as soon as someMethod() has been called 2 times under 100 ms
         verify(mock, timeout(100).atLeast(2)).someMethod();
```

#### after

After

验证是否在接下来指定时间内达到了指定的调用次数

```java
//passes after 100ms, if someMethod() has only been called once at that time.
         verify(mock, after(100)).someMethod();
         //above is an alias to:
         verify(mock, after(100).times(1)).someMethod();
      
         //passes if someMethod() is called *exactly* 2 times, as tested after 100 millis
         verify(mock, after(100).times(2)).someMethod();
      
         //passes if someMethod() has not been called, as tested after 100 millis
         verify(mock, after(100).never()).someMethod();
      
         //verifies someMethod() after a given time span using given verification mode
         //useful only if you have your own custom verification modes.
         verify(mock, new After(100, yourOwnVerificationMode)).someMethod();
```

#### timeout和after的区别

after要等到过了指定的时长后才能判断，timeout不一定必须等到过了指定的时长就能判断。

```java
//1.
mock.foo();
verify(mock, after(1000)).foo();
//waits 1000 millis and succeeds

//2.
mock.foo();
verify(mock, timeout(1000)).foo();
//succeeds immediately
```



### 顺序

InOrder

```java
List mockedList = mock(List.class);

mockedList.add("two");
mockedList.add("one");

InOrder inOrder = inOrder(mockedList);

inOrder.verify(mockedList).add("one");
inOrder.verify(mockedList).add("two");
```

## MockitoJUnitRunner


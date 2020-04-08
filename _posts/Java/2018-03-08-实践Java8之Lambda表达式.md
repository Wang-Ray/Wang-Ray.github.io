---
layout: post
title: "实践Java8之Lambda表达式"
date: 2018-02-25 11:08:00 +0800
categories: Java
tags: java Lambda
---
## Lambda

Lambda表达式（λ）本质上可以看做`匿名函数`，例如函数：

```java
public int add(int x, int y) {
	return x + y;
}
```

转换成Lambda表达式：

```java
(int x, int y) -> x + y;
// 参数类型有时也可以省略，如果Java编译器能够根据上下文推断出来
(x, y) -> x + y; 
// 显式指明返回值
(x, y) -> { return x + y; } 
```

可见Lambda表达式由三部分组成：`参数`，`箭头（->）`，以及一个`表达式`或`语句块`。

```java
// 没有参数，也没有返回值（比如：Runable的run方法）
() -> System.out.println("Hello Lambda!");
// 只有一个参数且参数类型可以省略时，参数括号可以不要
c -> { return c.size(); }
```

Java8加入了对Lambda表达式（λ）的支持，Lambda表达式的`目标类型（target type）`是`函数接口（function interface）`，适用于函数接口，详细使用参见[实践Java8之函数接口（@FunctionalInterface）](/java/2018/02/25/实践Java8之函数接口-@FunctioinInterface)

## Method References

有时候Lambda表达式仅仅是调用一个已经存在的方法（可以理解为方法已经有名字了，相比较于lambda是匿名函数），此时`method reference`可以简化调用（`::`），省略`参数`和`箭头`。

| Kind                                                         | Example                                |
| ------------------------------------------------------------ | -------------------------------------- |
| Reference to a `static method`                               | `ContainingClass::staticMethodName`    |
| Reference to an `instance method` of a particular object     | `containingObject::instanceMethodName` |
| Reference to an `instance method` of an arbitrary object of a particular type | `ContainingType::methodName`           |
| Reference to a `constructor`                                 | `ClassName::new`                       |

例如：
```java
// static method
button.setOnAction(System.out::println);
// 等价于如下lambda表达式
button.setOnAction(event->System.out.println(event));

// 
Arrays.sort(strings, String::compareToIgnoreCase);
```



## 参考

[Lambda](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)

[Java8 Lambda表达式教程](http://blog.csdn.net/ioriogami/article/details/12782141)

[Method References](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)
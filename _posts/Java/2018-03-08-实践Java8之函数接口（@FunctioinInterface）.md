---
layout: post
title: "实践Java8之函数接口（@FunctionalInterface）"
date: 2018-02-25 11:08:00 +0800
categories: Java
tags: java FunctionalInterface
---
## 介绍

一个接口，如果只有一个显式声明的抽象方法，那么它就是一个`函数接口`，Java8引入的新概念。

一般用注解`@FunctionalInterface`标注（非必须），例如：

```java
package java.lang;
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

Lambda表达式的`目标类型（target type）`是`函数接口（function interface）`

```java
Runnable r1 = () -> {System.out.println("Hello Lambda!");};
```

Lambda表达式也可以被看做是一个`Object`，前提是必须转型为一个`函数接口`，例如：

```java
// ERROR! Object is not a functional interface!
Object obj = () -> {System.out.println("Hello Lambda!");}; 
// 先转型为具体的函数接口
Runnable r1 = () -> {System.out.println("Hello Lambda!");};
Object obj = r1;
// 或者直接强制转换
Object o = (Runnable) () -> { System.out.println("hi"); };
```

JDK预定义了一些函数接口，在包java.util.function下，比如：`Runnable`、`Callable`、`Function`、`Supplier`、`Consumer`、`BiComsumer`和`Predicate`、`UnaryOperator`等

## 使用

### 基本用法

```java
// 嵌套的λ表达式
Callable<Runnable> c1 = () -> () -> { System.out.println("Nested lambda"); };
c1.call().run();
```

### 内部类

Lambda表达式主要用于替换以前广泛使用的内部匿名类，比如回调和Runnable等。

```java
// 传统
Thread oldSchool = new Thread( new Runnable () {
	@Override
    public void run() {
        System.out.println("This is from an anonymous class.");
    }
});
// 使用Lambda表达式
Thread gaoDuanDaQiShangDangCi = new Thread( () -> {
  	System.out.println("This is from an anonymous method (lambda exp).");
} );
```

### 集合类批处理操作

```java
// 传统
for(Object o: list) {
	System.out.println(o);
}
// 使用Lambda表达式
list.forEach(o -> {System.out.println(o);});
```



```java
// 挑出蓝色的，然后将这些蓝色的物体喷成红色
public void findBlueAndToRed(String... numbers) {
    List<Shape> shapes = Arrays.asList(numbers);
    shapes.stream()
      .filter(s -> s.getColor() == BLUE)
      .forEach(s -> s.setColor(RED));
}
```



```java
// 并行处理
shapes.parallelStream(); // 或shapes.stream().parallel()
```



```java
// 给出一个String类型的数组，找出其中所有不重复的素数
public void distinctPrimary(String... numbers) {
	List<String> l = Arrays.asList(numbers);
	List<Integer> r = l.stream()
      .map(e -> new Integer(e))
      .filter(e -> Primes.isPrime(e))
      .distinct()
      .collect(Collectors.toList());
	System.out.println("distinctPrimary result is: " + r);
}
```



```java
// 给出一个String类型的数组，找出其中各个素数，并统计其出现次数
public void primaryOccurrence(String... numbers) {
	List<String> l = Arrays.asList(numbers);
	Map<Integer, Integer> r = l.stream()
      .map(e -> new Integer(e))
      .filter(e -> Primes.isPrime(e))
      .collect( Collectors.groupingBy(p->p, Collectors.summingInt(p->1)) );
	System.out.println("primaryOccurrence result is: " + r);
}
```



```java
//给出一个String类型的数组，求其中所有不重复素数的和
public void distinctPrimarySum(String... numbers) {
	List<String> l = Arrays.asList(numbers);
    int sum = l.stream()
      .map(e -> new Integer(e))
      .filter(e -> Primes.isPrime(e))
      .distinct()
      .reduce(0, (x,y) -> x+y); // equivalent to .sum()
    System.out.println("distinctPrimarySum result is: " + sum);
}
```



```java
// 统计年龄在25-35岁的男女人数、比例
public void boysAndGirls(List<Person> persons) {
    Map<Integer, Integer> result = persons.parallelStream()
      .filter(p -> p.getAge()>=25 && p.getAge()<=35).
      collect(Collectors.groupingBy(p->p.getSex(), Collectors.summingInt(p->1)));
    System.out.print("boysAndGirls result is " + result);
    System.out.println(", ratio (male : female) is " +(float)result.get(Person.MALE)/result.get(Person.FEMALE));
}
```



### 函数式编程

`stream`：一个流通常以一个集合类实例为其数据源，然后在其上定义各种操作。

`pipeline`：流的API设计使用了管道（pipelines）模式，对流的一次操作会返回另一个流。

`lazy`：不会实时执行（管道贯通式）。

`eager`：实时执行，并且会触发lazy的执行。

forEach(Consumer)，map(Function)，filter(Predicate)，distinct()，toMap(Function, Function)，groupBy(Function, Collector)，

reduce()，sum()，max()，min()

## 常用函数接口

### Runnable

```java
/**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
```



### Callable

```java
/**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
```



### Function

```java
/**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
```

DoubleFunction、DoubleToIntFunction、DoubleToLongFunction、IntFunction、IntToDoubleFunction、IntToLongFunction、LongFunction

#### UnaryOperator

操作数和结果类型相同的Function，细分接口包括：DoubleUnaryOperator、IntUnaryOperator等

#### BiFunction

```java
/**
     * Applies this function to the given arguments.
     *
     * @param t the first function argument
     * @param u the second function argument
     * @return the function result
     */
    R apply(T t, U u);
```

##### BinaryOperator

操作数和结果相同类型的BiFunction，细分接口包括：IntBinaryOperator、DoubleBinaryOperator、LongBinaryOperator

### Supplier

```java
/**
     * Gets a result.
     *
     * @return a result
     */
    T get();
```

DoubleSupplier、BooleanSupplier、IntSupplier

### Consumer

```java
/**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
```

DoubleConsumer、IntConsumer、LongConsumer

#### BiConsumer

```java
/**
     * Performs this operation on the given arguments.
     *
     * @param t the first input argument
     * @param u the second input argument
     */
    void accept(T t, U u);
```

### Predicate

```java
/**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
```

DoublePredicate、IntPredicate、

#### BiPredicate

```java
/**
     * Evaluates this predicate on the given arguments.
     *
     * @param t the first input argument
     * @param u the second input argument
     * @return {@code true} if the input arguments match the predicate,
     * otherwise {@code false}
     */
    boolean test(T t, U u);
```



### UnaryOperator

## 参考

[Java8 Lambda表达式教程](http://blog.csdn.net/ioriogami/article/details/12782141)
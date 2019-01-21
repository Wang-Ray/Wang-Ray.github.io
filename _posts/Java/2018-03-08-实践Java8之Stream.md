---
layout: post
title: "实践Java8之Stream"
date: 2018-02-25 11:08:00 +0800
categories: Java
tags: java pipeline stream
---

Java8新增的Stream是对Collections的一个增强，专注于对集合对象的便利和高效的聚合操作，不仅支持串行的操作功能，而且还借助Java7中的Fork-Join机制支持了并行模式。

一个Stream通常是以一个集合类实例为其数据源，然后在其上定义各种操作。对 Stream 的使用就是实现一个 filter-map-reduce 过程,产生一个最终结果,或者导致一个副作用(side effect)。流的API设计使用了管道（pipelines）模式，对流的一次操作会返回另一个流。

## 获取Stream

## Stream分类

* Intermediate

  一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流,做出某种程度的数据映射/过滤,然后返回一个新的流,交给下一个操作使用。这类操作都是`惰性化的（lazy）`,就是说,仅仅调用到这类方法,并没有真正开始流的遍历。

* terminal

  一个流只能有一个 terminal 操作,当这个操作执行后,流就被使用“光”了,无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行,才会真正开始流的遍历,并且会生成一个结果,或者一个 side effect。

## 操作Stream

### Intermediate

#### map(Function)

one to another

#### flatMap(Function)

one to Stream

#### mapToInt(Function)

#### toMap(Function, Function)

#### filter(Predicate)

#### distinct()

去重

#### sorted()

升序排列

#### sorted(Comparator)

自定义排序

#### peek(Consumer)

类似forEach，只是不中断流

#### limit(long)

#### skip(long)

#### parallel()

返回并行流

#### sequential()

返回串行流

#### unordered()

### Terminal

#### forEach(Consumer)

#### forEachOrdered(Consumer)

 toArray
 Reduce
 Collect
 Min
 Max
 Count
 anyMatch
 allMatch
 noneMatch
 findFirst
 findAny
 iterator

#### groupBy(Function, Collector)



flatmap(Function)

Stream

generate()

iterate()

peek()

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

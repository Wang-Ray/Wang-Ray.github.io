---
15layout: post
title: "实践Spring之Conditional"
date: 2017-09-05 10:37:13 +0800
categories: Java
tags: spring condition conditional
---

从Spring 4.0开始，引入了条件化的配置（适用于Java Config），只有在满足某些特定条件的情况下才生效。

## @Conditional注解

## Condition接口

```java
public interface Condition {
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

例如：

```java
public class WindowsCondition implements Condition{
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment().getProperty("os.name").contains("Windows");
    }
}
```



```java
@Bean
@Conditional(WindowsCondition.class)
public ListService windowsListService() {
    return new WindowsListService();
}
```

样例代码可以参见[github](https://github.com/AngiWANG/sample-conditional)

## SpringBootCondition接口



| 条件化注解                           | 配置生效条件                                   |
| ------------------------------- | ---------------------------------------- |
| @ConditionalOnBean              | 配置了某个特定Bean                              |
| @ConditionalOnMissingBean       | 没有配置特定的Bean                              |
| @ConditionalOnClass             | Classpath里有指定的类                          |
| @ConditionalOnMissingClass      | Classpath里缺少指定的类                         |
| @ConditionalOnExpression        | 给定的Spring Expression Language(SpEL)表达式计算结果为 true |
| @ConditionalOnJava              | Java的版本匹配特定值或者一个范围值                      |
| @ConditionalOnJndi              | 参数中给定的JNDI位置必须存在一个,如果没有给参数,则要有JNDI InitialContext |
| @ConditionalOnProperty          | 指定的配置属性要有一个明确的值                          |
| @ConditionalOnResource          | Classpath里有指定的资源                         |
| @ConditionalOnWebApplication    | 这是一个Web应用程序                              |
| @ConditionalOnNotWebApplication | 这不是一个Web应用程序                             |
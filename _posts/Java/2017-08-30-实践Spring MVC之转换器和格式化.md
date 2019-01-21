---
layout: post
title: "实践Spring MVC之转换器和格式化"
date: 2017-08-30 17:08:00 +0800
categories: Java
tags: spring-mvc validator jsr-303
---

Spring内置了很多转换器（Converter）和格式化（Formatter），能够完成常规的类型转换。



当类型转换失败，比如：product的productDate字段，输入“abc”转换为LocalDate，异常如下：

```
org.springframework.beans.TypeMismatchException: Failed to convert property value of type [java.lang.String] to required type [java.time.LocalDate] for property 'productionDate'; nested exception is org.springframework.core.convert.ConversionFailedException: Failed to convert from type [java.lang.String] to type [@javax.validation.constraints.NotNull @javax.validation.constraints.Past java.time.LocalDate] for value 'abc'; nested exception is java.lang.IllegalArgumentException: invalid date format. Please use this pattern"MM-dd-yyyy"
```

可以定制异常消息，避免讲上述内容暴露给用户，例如：

```properties
# 字段定制
typeMismatch.productionDate=Invalid production date
```

或者

```properties
# 类型定制
typeMismatch=Invalid input
```


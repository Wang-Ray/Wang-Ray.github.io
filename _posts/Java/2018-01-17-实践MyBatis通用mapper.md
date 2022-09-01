---
layout: post
title: "实践MyBatis通用mapper"
date: 2018-01-17 14:05:00 +0800
categories: Java
tags: database jpa java mybatis mapper
---

[通用mapper](https://mapperhelper.github.io)

```xml
<dependency>
	<groupId>tk.mybatis</groupId>
	<artifactId>mapper</artifactId>
	<version>3.5.0</version>
</dependency>
```

## mapper spring boot

```xml
<dependency>
	<groupId>tk.mybatis</groupId>
	<artifactId>mapper-spring-boot-starter</artifactId>
	<version>1.2.1</version>
</dependency>
```

## mapper for MyBatis Generator

为了配合通用mapper，对MyBatis Generator做了一些自定义的扩展，主要包括：

* MapperCommentGenerator

自定义注释生成器

* useMapperCommentGenerator

给`<context>`增加`<property>`：useMapperCommentGenerator，默认值是true，表示使用自定义注释生成器`MapperCommentGenerator`

* MapperPlugin

```xml
<plugin type="tk.mybatis.mapper.generator.MapperPlugin">
	<property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
</plugin>
```



| 属性                 | 描述                                 | 备注                                       |
| ------------------ | ---------------------------------- | ---------------------------------------- |
| caseSensitive      | 默认值：false                          |                                          |
| forceAnnotation    | 是否强制生成注解（@Table，@Column），默认值：false | 也用于注释生成器，依赖`<context>`的`<property>`之useMapperCommentGenerator |
| beginningDelimiter | 默认值：空串                             | 同上，作用于@Table和@Column                     |
| endingDelimiter    | 默认值：空串                             | 同上，作用于@Table和@Column                     |
| schema             | 数据库schema，默认值：null                 | 表名前缀，作用于@Table                           |
|                    |                                    |                                          |


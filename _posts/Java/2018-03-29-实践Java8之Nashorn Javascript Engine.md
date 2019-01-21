---
layout: post
title: "实践Java8之Nashorn Javascript Engine"
date: 2018-03-29 11:08:00 +0800
categories: Java
tags: java nashorn javascript
---

Nashorn是一个新的JavaScript引擎（Java6引入过一个JavaScript引擎Rhino），性能更好，对ECMAScript标准有极强的兼容性。

命令行运行nashorn

```shell
$ jjs
jjs> 'Hello, World!'.length
12
jjs>
```

Java中运行nashorn

```java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("nashorn");
// 执行js
Object result = engine.eval("'Hello, World!'.length");
System.out.println(result);
// 执行外部js
result = engine.eval(Files.newBufferedReader(Paths.get("hello.js")));
System.out.println(result);
```
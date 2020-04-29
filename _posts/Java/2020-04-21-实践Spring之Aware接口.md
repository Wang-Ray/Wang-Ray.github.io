---
layout: post
categories: Java
tags: Java Spring Aware
---

`Aware`：翻译过来是知道的，已感知的，意识到的，所以这些`*Aware`接口从字面意思应该是能感知到所有`Aware`前面的含义。

比如ApplicationContextAware、BeanFactoryAware和BeanNameAware等

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods
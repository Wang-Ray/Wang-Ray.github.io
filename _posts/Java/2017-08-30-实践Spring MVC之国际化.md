---
layout: post
title: "实践Spring MVC之国际化"
date: 2017-08-30 17:08:00 +0800
categories: Java
tags: spring-mvc i18n international locale
---

基于LocaleResolver取得Locale，参考实现包括如下三种：

1. AcceptHeaderLocaleResolver

   基于客户端浏览器语言设置

2. SessionLocaleResolver

   基于session

3. CookieLocaleResolver

   基于cookie

```xml
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver">
</bean>
```



页面中使用标签<spring:message/>，例如：

```jsp
<spring:message code="label.productName" text="default text" />
```


---
layout: post
title: "实践Spring MVC"
date: 2017-08-22 17:08:00 +0800
categories: Java
tags: spring-mvc
---

[Spring MVC](http://projects.spring.io/spring-framework/)是Spring Framework的Web解决方案之一。

## 前端控制器（DispatcherServlet）

## RequestMapping

### 处理器映射（HandlerMapping）

请求到处理器（以及拦截器）的映射，默认实现：**BeanNameUrlHandlerMapping**，uri到到同名bean的映射，比如URI"/hello"映射到bean"/hello"。

**DefaultAnnotationHandlerMapping**：2.5新增注解式控制器支持

**RequestMappingHandlerMapping**：3.1新增替代2.5，支持更多扩展点

### 处理器适配器（HandlerAdapter）

适配支持不同的处理器，默认实现：**SimpleControllerHandlerAdapter**，实现Controller接口的bean作为处理器。

**AnnotationMethodHandlerAdapter**：2.5新增注解式控制器支持

**RequestMappingHandlerAdapter**：3.1新增替代2.5，支持更多扩展点

`<mvc:annotation-driven/>`或	@EnableWebMvc

### 静态资源映射

## 处理器或页面控制器（Controller）

![Controller Hierarchy](/images/controller-hierarchy.png)

请求处理方法（request-handling method）：控制器中处理http请求的方法，annotated handler method

### ViewController

## 拦截器（HandlerInterceptor）



## ViewResolver

根据逻辑视图名解析具体视图，默认实现：**BeanNameViewResolver**，根据逻辑视图名找同名bean（实现View接口）进行渲染

**InternalResourceViewResolver**，支持jsp视图

**ContentNegotiatingViewResolver**，ViewResolver代理，委托给不同的ViewResolver来处理不同的View

### RequestToViewNameTranslator

## ModelAndView

### Model

数据

### View

对具体视图页面的封装，比如jsp/freemarker/velocity/thymeleaf/beetl/handlebars/mustache/rocker

## HttpMessageConverter

http输入/输出转换，比如JSON或XML等的数据输入/输出转换，一般用于@RequestBody和@ResponseBody，3.0开始，只需相应的jar（比如Jackson（JSON）或JAXP（XML））存在于classpath，会自动注册





## 数据绑定（Data Binder）

表单数据或URL请求参数（均为字符串）自动绑定到**命令对象**，会自动进行类型**转换**或**格式化**（内置或自定义组件），以及手工加入的**验证**。

BindingResult：绑定结果（包含转换、格式化和验证），功能方法添加其作为入参，如果有错误，页面可以进行显示（`<form:errros/>`），例如：

```java
@RequestMapping(value = "/save-book")
public String saveBook(@ModelAttribute Book book, BindingResult bindingResult) {
    // 绑定转换校验失败时页面提示
	if (bindingResult.hasErrors()) {
        FieldError fieldError = bindingResult.getFieldError();
        logger.info("Code:" + fieldError.getCode() + ", object:"
                + fieldError.getObjectName() + ", field:"
                + fieldError.getField());
        return "BookAddForm";
    }
    
    Category category = bookService.getCategory(book.getCategory().getId());
    book.setCategory(category);
    bookService.save(book);
    return "redirect:/list-books";
}
```

```xml
<p class="errorLine">
	<form:errors path="price" cssClass="error"/>
</p>
```



### 命令对象（Command）

封装请求参数的DTO，比如表单数据

表单对象（form object）

### 转换器（Converter）

一种类型转换为另一种类型，适用于所有层

### 格式化（Formatter）

String与其他类型的转换，较为适用于Spring MVC

### 验证器（Validator）

JSR-303 Bean Validation 1.0

JSR-349 Bean Validation 1.1

FlashMapManager

ExceptionHandlerExceptionResolver

### 国际化（LocaleResolver）

主题（ThemeResolver）

文件上传和文件下载（ThemeResovler）

Spring表达式（Spring EL）

### Spring form标签库（form tag）

借助于form tag，可以自动实现表单填充，比如**初始表单页面预填充**，**表单校验失败后表单填充**

JSTL

单元测试和集成测试

## HandlerExceptionResolver


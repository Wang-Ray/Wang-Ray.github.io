---
layout: post
title: "实践Spring MVC之文件上传下载"
date: 2017-08-30 17:08:00 +0800
categories: Java
tags: spring-mvc upload download
---

Spring MVC文件上传基于MultipartResolver实现，参考实现包括如下两种：

## Apache Commons FileUpload

CommonsMultipartResolver

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<property name="maxUploadSize" value="2000000"/>
</bean>
```

## Serlvet 3

StandardServletMultipartResolver，定义bean`multipartResolver`，例如：

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver">
</bean>
```

同时还需要对相应Servlet进行文件上传相关配置，有下面几种方式：

### web.xml中Servlet新增`<multipart-config/>`

```xml
<servlet>
	<servlet-name>springmvc</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/config/springmvc-config.xml</param-value>
	</init-param>
	<multipart-config>
		<location>/app/upload</location>
		<max-file-size>20848820</max-file-size>
		<max-request-size>418018841</max-request-size>
		<file-size-threshold>1048576</file-size-threshold>
	</multipart-config>
</servlet>
```

​

### 编程式Servlet注册方式下，注入MultipartConfigElement实例

```java
//代替web.xml
public class WebInitializer implements WebApplicationInitializer {//1

	@Override
	public void onStartup(ServletContext servletContext)
			throws ServletException {
		AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(MyMvcConfig.class);
        ctx.setServletContext(servletContext); //2
        
        Dynamic servlet = servletContext.addServlet("dispatcher", new DispatcherServlet(ctx)); //3
        servlet.addMapping("/");
        servlet.setMultipartConfig(new MultipartConfigElement("/app/upload",20848820,418018841,1048576));
        servlet.setLoadOnStartup(1);
        servlet.setAsyncSupported(true);//1
	}
}
```

​

### 在注解式Servlet上使用MultipartConfig注解

### 在spring boot中，定义MultipartConfigElement的bean即可。

```java
	@Bean
    public MultipartConfigElement multipartConfigElement() {
        MultipartConfigFactory factory = new MultipartConfigFactory();
        // 设置文件大小限制 ,超出设置页面会抛出异常信息，
        // 这样在文件上传的地方就需要进行异常信息的处理了;
        factory.setMaxFileSize("256KB"); // KB,MB
        /// 设置总上传数据总大小
        factory.setMaxRequestSize("512KB");
        // Sets the directory location where files will be stored.
        factory.setLocation("路径地址");
        return factory.createMultipartConfig();
    }
```

## Form设置

```html
<form enctype="multipart/form-data"> 
</form>
```



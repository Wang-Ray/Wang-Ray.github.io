---
layout: post
title: "实践Async Spring MVC"
date: 2018-05-09 11:08:00 +0800
categories: Java
tags: java Spring-MVC Async
---



## 样例一

web.xml

```xml
<?xml version="1.0" encoding="ASCII"?>
<web-app version="3.1" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee.http://java.sun.com/xml/ns/javaee/web-app_3_1.xsd">
<servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/mvc-config.xml</param-value>
        </init-param>
        <async-supported>true</async-supported>
        <load-on-startup>1</load-on-startup>
    </servlet>
</web-app>
```

HelloWorldController.java

```java
@Controller
public class HelloWorldController {

    Logger logger = LoggerFactory.getLogger(HelloWorldController.class);

    @RequestMapping("/helloWorld")
    @ResponseBody
    public DeferredResult<String> helloWorld() {
        logger.info("helloWorld begin");
      
      	try {
			TimeUnit.SECONDS.sleep(3);
		} catch (Exception e) {
			e.printStackTrace();
		}
      
        final DeferredResult<String> deferredResult = new DeferredResult<String>(30 * 1000);
        // 另开worker线程进行回调处理
        deferredResult.onCompletion(new Runnable() {
            public void run() {
                logger.info("onCompletion begin");
                try {
                    Thread.sleep(10 * 1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                logger.info("onCompletion end");
            }

        });
        // 自定义线程进行业务处理
        new Thread(new Runnable() {
            public void run() {
                logger.info("biz begin");
                try {
                    Thread.sleep(10 * 1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                deferredResult.setResult("Hello, World!");
                logger.info("biz end");
            }

        }).start();
        logger.info("helloWorld end");
        return deferredResult;
    }
}
```

收到一笔请求时日志输出：

```
2018-05-09 12:03:06.947 INFO [http-bio-8080-exec-1] HelloWorldController:21 >> helloWorld begin
2018-05-09 12:03:06.949 INFO [http-bio-8080-exec-1] HelloWorldController:50 >> helloWorld end
2018-05-09 12:03:06.949 INFO [Thread-4] HelloWorldController$2:39 >> biz begin
2018-05-09 12:03:16.951 INFO [Thread-4] HelloWorldController$2:46 >> biz end
2018-05-09 12:03:16.972 INFO [http-bio-8080-exec-2] HelloWorldController$1:26 >> onCompletion begin
2018-05-09 12:03:26.973 INFO [http-bio-8080-exec-2] HelloWorldController$1:32 >> onCompletion end
```

（设定worker线程池大小为3的情况下）连续收到两笔请求时线程栈如下：

![AsyncServlet](/images/AsyncServlet.png)

可见worker线程在AysncServlet下不是一直占用至Servlet结束，这样一个worker线程可以服务于多个请求。

## 样例二

```java
@Controller
public class ShowMessageController {

	Logger logger = LoggerFactory.getLogger(ShowMessageController.class);
	
	@RequestMapping("/showMessage")
	public DeferredResult<ModelAndView> showMessage() {
		final DeferredResult<ModelAndView> deferredResult = new DeferredResult<ModelAndView>(30 * 1000);

		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					Thread.sleep(10 * 1000);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				ModelAndView modelAndView = new ModelAndView();
				modelAndView.setViewName("showMessage");
				modelAndView.addObject("message", "Hello, World!");
				deferredResult.setResult(modelAndView);
			}

		}).start();
		return deferredResult;
	}
}
```

## DeferredResult



```java
public DeferredResult(Long timeout, Object timeoutResult) {
	this.timeoutResult = timeoutResult;
	this.timeout = timeout;
}
```

默认`timeout`是null，`timeoutResult`是`new Object()`

## setResult(T result)

默认`result`是`new Object()`

## setErrorResult(Object result)

## onComplete

## onTimeout

当没有注册`onTimeout`时，默认超时后会应答500

```
HTTP Status 500 -

type Status report

message

description The server encountered an internal error that prevented it from fulfilling this request.
Apache Tomcat/7.0.78
```


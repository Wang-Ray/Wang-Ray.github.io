---
layout: post
title: "实践Spring之IoC或DI"
date: 2017-07-28 09:25:00 +0800
categories: Java
tags: spring IoC DI
---

面向对象编程中很重要的一步就是进行对象设计和定义（比如类或接口等），而下一步就要考虑如何使用这些对象，一般是先通过实例化对象来生产实例，然后调用实例的方法来执行逻辑。其中实例化有多种方法，传统的new，Class.forName()和反射等，当项目较大，对象和实例将会非常多，依赖关系也会较为复杂，一般传统的方式会较难控制。Spring提供的IoC或DI，使用配置的方式来完成**实例容器**、**实例化**、**实例间的依赖关系（依赖注入）**和**生命周期管理**等。

auto-detect

autowire

## 实例容器

### BeanFactory

#### BeanFactoryAware

### ApplicationContext

常用的有：

ClassPathXmlApplicationContext

AnnotationConfigApplicationContext

AnnotationConfigWebApplicationContext：适用于web项目

BeanFactoryPostProcessor

实例化bean的方法：实例工厂、静态工厂和构造器



#### ApplicationContextAware

### BeanPostProcessor

```
InstantiationAwareBeanPostProcessor
DestructionAwareBeanPostProcessor
```

### BeanFactoryPostProcessor

```
PropertyResourceConfigurer
```

## 基于XML

### `<import/>`

### `<context:component-scan/>`

### `<context:annotation-config/>`

## 基于注解

### Bean定义注解

以下注解被扫描后实现bean定义

#### @Component

会自动constructor注入

```java
@Component
public class CityDao {

    private SqlSession sqlSession;

    public CityDao(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }
}
```

不会自动setter注入

```java
@Component
public class CityDao {

    private SqlSession sqlSession;

    public SqlSession getSqlSession() {
        return sqlSession;
    }

    public void setSqlSession(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }
}
```



#### @Controller

#### @Service

#### @Repository

#### @Entity

### 依赖注入注解

#### @Autowired

Spring4新增泛型注入支持，例如：

```java
public abstract class BaseService<T> implements IService<T> {

    @Autowired
    protected Mapper<T> mapper;
...
}
```

## 基于Java Config

### @Configuration

定义@bean的载体，好比一个定义`<bean>`的xml文件，可以被实例容器加载

### @Import

引入其他的@Configuration，有如下三种方式

1. 直接引入

   直接引入另外一个配置类，最常用的方式，简单明了，例如：

   `@Import(App2Config.class)`，效果类似`<import/>`

2. 条件引入

   通过org.springframework.context.annotation.**ImportSelector**实现，实例参见[实践Spring Enable系列之EnableAsync](/2017/08/05/实践Spring-Enable系列之EnableAsync.html)

3. 动态注册

   通过org.springframework.context.annotation.**ImportBeanDefinitionRegistrar**实现，实例参见[实践Spring-Enable系列之EnableAspectJAutoProxy](/2017/08/05/实践Spring-Enable系列之EnableAspectJAutoProxy.html)

### @Bean

bean定义

### @Scope

默认是单例（singleton），也可以指定为原型（prototype），

```java
@Scope("singleton")
@Scope("prototype")
```

### @Primary

首选bean

### @Qulifier

指定注入那个bean

### 依赖注入

1. 引用方法注入

2. 方法参数注入

   默认按照`类型注入`，当有多个同类型实例时按照`名字注入`，如果注入失败则报存在多个同类型实例错误。

   1. 构造方法参数注入
   2. @Bean方法参数注入

详情参见下面样例

### 生命周期

```java
@Bean(initMethod = "init", destroyMethod = "destory")
```

*备注：需要实例容器ctx.registerShutdownHook()，否则destoryMethod不会触发*

### 样例

SaveVideoInfoDao.java

```java
public class SaveVideoInfoDao{
  	public String hello(String name) {
		return "hello, " + name;
	}
	public void init() {
		System.out.println("init SaveVideoInfoDao instance");
	}

	public void destory() {
		System.out.println("destory SaveVideoInfoDao instance");
	}
}
```

VideoService.java

```java
public class VideoService{
  	private SaveVideoInfoDao saveVideoInfoDao;
	public VideoService(SaveVideoInfoDao saveVideoInfoDao){
    	this.saveVideoInfoDao = saveVideoInfoDao;
	}
  	public void sayHello(String name) {
		System.out.println(saveVideoInfoDao.hello(name));
	}
}
```

App1Config.java

```java
@Configuration
@Import(App2Config.class)
public class App1Config{
	@Bean(initMethod = "init", destroyMethod = "destory")
    public SaveVideoInfoDao saveVideoInfoDao() {
        return new SaveVideoInfoDao();
    }
    //引用方法注入
    @Bean
    public VideoService videoService1() {
        VideoService videoService = new VideoService(saveVideoInfoDao());
        return videoService;
    }
}
```

App2Config.java

```java
@Configuration
public class App2Config{
	@Bean(initMethod = "init", destroyMethod = "destory")
    public SaveVideoInfoDao saveVideoInfoDao() {
        return new SaveVideoInfoDao();
    }
    //方法参数注入
    @Bean
    public VideoService videoService2(SaveVideoInfoDao saveVideoInfoDao) {
        VideoService videoService = new VideoService(saveVideoInfoDao);
        return videoService;
    }
}
```
App3Config.java

```java
@Configuration
public class App3Config {
	
	private ISaveVideoInfoDao saveVideoInfoDao;
	
	// 构造参数注入
	public App3Config(ISaveVideoInfoDao saveVideoInfoDao){
		this.saveVideoInfoDao = saveVideoInfoDao;
	}

	@Bean
	public IVideoService videoService3() {
		IVideoService videoService = new VideoService(saveVideoInfoDao);
		return videoService;
	}
}
```

SampleMain.java

```java
public class SampleMain{
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(App1Config.class);
        ctx.registerShutdownHook();
        VideoService videoService1 = ctx.getBean("videoService1", VideoService.class);
        videoService1.sayHello("abc");
        VideoService videoService2 = ctx.getBean("videoService2", VideoService.class);
        videoService2.sayHello("def");
        VideoService videoService3 = ctx.getBean("videoService3", VideoService.class);
	    videoService3.sayHello("ghi");
    }
}
```

样例源码请移步[github之sample-spring-javaconfig](https://github.com/AngiWANG/sample-spring-javaconfig)

### @ComponentScan

Configures component scanning directives for use with @`Configuration` classes. Provides support parallel with Spring XML's `<context:component-scan/>` element.

One of `basePackageClasses()`, `basePackages()` or its alias `value()` may be specified to define specific packages to scan. `If specific packages are not defined scanning will occur from the package of the class with this annotation`.

Note that the `<context:component-scan>` element has an `annotation-config` attribute, however this annotation does not. This is because in almost all cases when using `@ComponentScan`, default annotation config processing (e.g. processing `@Autowired` and friends) is assumed. Furthermore, when using `AnnotationConfigApplicationContext`, annotation config processors are always registered, meaning that any attempt to disable them at the `@ComponentScan` level would be ignored.

### @ImportResource

```java
@ImportResource({"classpath:abc-context.xml","classpath:def-context.xml"})
```


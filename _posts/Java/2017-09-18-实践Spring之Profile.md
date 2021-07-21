---
layout: post
title: "实践Spring之Profile"
date: 2017-09-18 17:08:00 +0800
categories: Java
tags: spring profile
---

Spring 3.1引入了Profile，是一种条件化配置，基于运行时激活的Profile，实现动态配置，会使用或忽略不同的配置类、Bean、属性文件和属性等。一般用于不同的环境下使用不同的配置。激活方法包括：

1. 设置spring.profiles.active属性可以激活相应的Profile
2. Environment.setActiveProfiles("prod")
3. @Profile("prod")

## 应用于配置类的Profile

```java
@Profile("production")
@Configuration
public class SampleConfig {
...
}
```
## 应用于Bean的Profile

```java
@Configuration
public class SampleConfig {
...
	@Bean
	@Profile("dev")
	public DemoBean devDemoBean(){
    	return new DemoBean("for develeopment profile");
	}
  
  	@Bean
	@Profile("prod")
	public DemoBean prodDemoBean(){
    	return new DemoBean("for production profile");
	}
...
}
```

## 应用于配置文件（Properties）的Profile

application.properties

application-dev.properties

application-prod.properties

## 应用于配置文件（YAML）的Profile

application.yaml

application-dev.yaml

application-prod.yaml

## 应用于属性（YAML）的Profile

```yaml
# 默认配置
logging:
  level:
    root: INFO

---
# 开发环境配置
spring:
  profiles: development
logging:
  level:
    root: DEBUG

---
# 生产环境配置
spring:
  profiles: production
logging:
  path: /tmp/
  file: BookWorm.log
  level:
    root: WARN
```

## 应用于Logback

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml" />
    <logger name="org.springframework.web" level="INFO"/>

    <springProfile name="default">
        <logger name="org.springboot.sample" level="TRACE" />
    </springProfile>

    <springProfile name="dev">
        <logger name="org.springboot.sample" level="DEBUG" />
    </springProfile>

    <springProfile name="staging">
        <logger name="org.springboot.sample" level="INFO" />
    </springProfile>

</configuration>
```


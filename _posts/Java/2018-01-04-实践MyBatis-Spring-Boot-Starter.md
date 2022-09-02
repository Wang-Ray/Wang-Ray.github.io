---
layout: post
title: "实践MyBatis-Spring-Boot-Starter"
date: 2018-01-04 10:40:00 +0800
categories: Java
tags: database MyBatis MyBatis-Spring-Boot-Starter JDBC ORM
---

[MyBatis-Spring-Boot-Starter](http://www.mybatis.org/spring-boot-starter/)，有助于快速构建基于MyBatis的Spring Boot应用。

MyBatis自动配置类`MybatisAutoConfiguration`

自动发现DataSource

创建并注册一个`SqlSessionFactory`实例

创建并注册一个`SqlSessionTemplate`实例

自动扫描@Mapper注解的接口为Mapper，可以通过@MapperScan自定义扫描

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0-SNAPSHOT</version>
</dependency>
```



```java
@Mapper
public interface CityMapper {
	// annotation mapper
    @Select("SELECT * FROM CITY WHERE state = #{state}")
    City findByState(@Param("state") String state);

}
```



```java
@SpringBootApplication
public class SampleMybatisApplication implements CommandLineRunner {

    private final CityMapper cityMapper;

    public SampleMybatisApplication(CityMapper cityMapper) {
        this.cityMapper = cityMapper;
    }

    public static void main(String[] args) {
        SpringApplication.run(SampleMybatisApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println(this.cityMapper.findByState("CA"));
    }

}
```



### 配置

对应配置类：`MybatisProperties`，mybatis前缀

| Property                   | Description                              |
| -------------------------- | ---------------------------------------- |
| `config-location`          | Location of MyBatis xml config file.     |
| `check-config-location`    | Indicates whether perform presence check of the MyBatis xml config file. |
| `mapper-locations`         | Locations of Mapper xml config file.【支持通配符】 |
| `type-aliases-package`     | Packages to search for type aliases. (Package delimiters are "`,; \t\n`") |
| `type-handlers-package`    | Packages to search for type handlers. (Package delimiters are "`,; \t\n`") |
| `executor-type`            | Executor type: `SIMPLE`, `REUSE`, `BATCH`. |
| `configuration-properties` | Externalized properties for MyBatis configuration. Specified properties can be used as placeholder on MyBatis config file and Mapper file.		      For detail see the [MyBatis reference page](http://www.mybatis.org/mybatis-3/configuration.html#properties) |
| `configuration`            | A MyBatis `Configuration` bean. About available properties see the [MyBatis reference page](http://www.mybatis.org/mybatis-3/configuration.html#settings).		      NOTE This property cannot be used at the same time with the `config-location`. |

mybatis.configuration与mybatis.config-location不能同时使用

7个非mybatis.configuration属性

49个mybatis.configuration属性

```properties
# application.properties
mybatis.type-aliases-package=com.example.domain.model
mybatis.type-handlers-package=com.example.typehandler
mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.default-fetch-size=100
mybatis.configuration.default-statement-timeout=30
mybatis.mapper-locations=classpath*:**/mapper/*.xml
```

### ConfigurationCustomizer

```java
// @Configuration class
@Bean
ConfigurationCustomizer mybatisConfigurationCustomizer() {
    return new ConfigurationCustomizer() {
        @Override
        public void customize(Configuration configuration) {
            // customize ...
        }
    };
}
```

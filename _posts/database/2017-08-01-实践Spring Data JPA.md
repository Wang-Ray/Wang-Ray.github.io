---
layout: post
title: "实践Spring Data JPA"
date: 2017-08-01 10:27:00 +0800
categories: 数据库
tags: database spring-data spring-data-jpa
---

[Spring Data JPA](http://projects.spring.io/spring-data-jpa), part of the larger [Spring Data](http://projects.spring.io/spring-data) family, makes it easy to easily implement JPA based repositories. This module deals with enhanced support for JPA based data access layers. It makes it easier to build Spring-powered applications that use data access technologies.

相比较于原始JPA或Spring JPA，Spring Data JPA封装和扩展更进了一步。基于XML和JavaConfig方式参考如下：

## JPA xml namespace

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:jpa="http://www.springframework.org/schema/data/jpa"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/data/jpa
http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">
	<!-- repository所在包 -->
  	<jpa:repositories base-package="com.acme.repositories"/>
</beans>
```

## JPA JavaConfig

```java
@Configuration
@EnableJpaRepositories(basePackages="spittr.db")
@EnableTransactionManagement
class ApplicationConfig {
    @Bean
    public DataSource dataSource() {
        EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
        return builder.setType(EmbeddedDatabaseType.HSQL).build();
    }
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        vendorAdapter.setGenerateDdl(true);
        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setJpaVendorAdapter(vendorAdapter);
        factory.setPackagesToScan("com.acme.domain");
        factory.setDataSource(dataSource());
        return factory;
    }
    @Bean
    public PlatformTransactionManager transactionManager() {
        JpaTransactionManager txManager = new JpaTransactionManager();
        txManager.setEntityManagerFactory(entityManagerFactory());
        return txManager;
    }
}
```

## Jpa Repositories

![spring-data-JpaRepository](/images/spring-data-JpaRepository.png)

定义泛型参数化的继承Spring Data JPA的某一个接口，自动生成实现（`org.springframework.data.jpa.repository.support.SimpleJpaRepository<T, ID>`），拥有18个基本方法

```java
<SpringDataJpaRepository接口> extends JpaRepository<Model, Long>
```


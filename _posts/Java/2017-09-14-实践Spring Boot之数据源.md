---
layout: post
title: "实践Spring Boot之数据源"
date: 2017-09-14 17:08:00 +0800
categories: Java
tags: spring spring-boot datasource
---

基于数据库的Java应用涉及两个概念，一个是使用哪种**数据库（Database）**，比如：Oracle、MySQL和嵌入式数据库（比如：h2/derby/hsql）等，另一个是使用哪种**数据源（DataSource，也即数据库连接池（connection pool））**，比如：tomcat数据源/Hikari/DBCP1/DBCP2/SimpleDriverDataSource等，数据库和数据源可以任意搭配。

spring boot自动配置支持多种数据源（DataSourceAutoConfiguration），比如：**Tomcat数据源/Hikari/DBCP1/DBCP2连接池和SimpleDriverDataSource**，根据相应class（DataSourcePoolMetadataProvidersConfiguration）或spring.datasource.type（DataSourceProperties）属性的取值来判断，可取值包括如下：

* org.apache.tomcat.jdbc.pool.DataSource
* com.zaxxer.hikari.HikariDataSource
* org.apache.commons.dbcp.BasicDataSource
* org.apache.commons.dbcp2.BasicDataSource

**嵌入式数据库数据源**默认使用嵌入式数据库（EmbeddedDatabase）和**SimpleDriverDataSource**数据源（spring-jdbc提供），比其他数据源优先级低。

spring boot 1.x 默认数据源是tomcat数据源，spring boot 2.x 默认数据源是Hikari

## 通用配置

配置类：org.springframework.boot.autoconfigure.jdbc.DataSourceProperties，配置前缀：spring.datasource

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost/readinglist
    username: dbuser
    password: dbpass
```

可以不用指定驱动名称，spring boot会自动根据url识别。

如果url也不指定，则根据已加载的驱动识别使用嵌入式数据库（EmbeddedDatabaseConnection）。

```java
	public String determineUrl() {
		if (StringUtils.hasText(this.url)) {
			return this.url;
		}
		String url = this.embeddedDatabaseConnection.getUrl(determineDatabaseName());
		if (!StringUtils.hasText(url)) {
			throw new DataSourceBeanCreationException(this.embeddedDatabaseConnection,
					this.environment, "url");
		}
		return url;
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		this.embeddedDatabaseConnection = EmbeddedDatabaseConnection
				.get(this.classLoader);
	}
```



具体数据源的定义以及相关配置由org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration实现。

## Tomcat数据源

spring-boot-starter-data-jpa和spring-boot-starter-jdbc默认使用Tomcat数据源

```xml
<dependency>
	<groupId>org.apache.tomcat</groupId>
	<artifactId>tomcat-jdbc</artifactId>
</dependency>
```

配置类：org.apache.tomcat.jdbc.pool.DataSource

## DBCP2数据源

### 替换为DBCP2

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
    <exclusions>
		<exclusion>
			<groupId>org.apache.tomcat</groupId>
			<artifactId>tomcat-jdbc</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-dbcp2</artifactId>
</dependency>
```

配置类：org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration.Dbcp2，配置前缀：spring.datasource.dbcp2，例如：

```yaml
spring:
  datasource:
    dbcp2:
      testOnBorrow: false
      testOnReturn: false
```

## 嵌入式数据库数据源

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
    <exclusions>
		<exclusion>
			<groupId>org.apache.tomcat</groupId>
			<artifactId>tomcat-jdbc</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

org.springframework.boot.autoconfigure.jdbc.EmbeddedDataSourceConfiguration



配置类：SimpleDriverDataSource和EmbeddedDatabaseBuilder

默认HSQL，默认脚本schema.sql和data.sql

```
2017-09-15 21:40:40.122  INFO 4478 --- [           main] o.s.j.d.e.EmbeddedDatabaseFactory        : Starting embedded database: url='jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=false', username='sa'

```



## 测试数据库自动配置

针对一次性测试的数据库需求，spring boot提供了特定支持，自动配置类为TestDatabaseAutoConfiguration。

```java
@Configuration
@AutoConfigureBefore(DataSourceAutoConfiguration.class)
public class TestDatabaseAutoConfiguration {
...
	@Bean
	@ConditionalOnProperty(prefix = "spring.test.database", name = "replace", havingValue = "AUTO_CONFIGURED")
	@ConditionalOnMissingBean
	public DataSource dataSource() {
		return new EmbeddedDataSourceFactory(this.environment).getEmbeddedDatabase();
	}
...
}
```



开发测试时**嵌入式数据库**可能还不错，spring-boot的支持也相当给力，可以不用做任何配置。

对于生产等环境，需要使用独立的数据库服务器，则可以通过配置即可实现。
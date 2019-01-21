---
layout: post
title: "实践Spring Data"
date: 2017-07-26 17:14:00 +0800
categories: 数据库
tags: database nosql spring-data
---

[Spring Data](http://projects.spring.io/spring-data/)’s mission is to provide a familiar and consistent, Spring-based programming model for **data access** while still retaining the special traits of the underlying data store.
It makes it easy to use data access technologies, **relational** and **non-relational** databases, **map-reduce** frameworks, and **cloud-based** data services. This is an umbrella project which contains many subprojects that are specific to a given database. The projects are developed by working together with many of the companies and developers that are behind these exciting technologies.

**Spring Data Release Train - BOM**

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-releasetrain</artifactId>
      <version>${release-train}</version>
      <scope>import</scope>
      <type>pom</type>
    </dependency>
  </dependencies>
</dependencyManagement>
```

*当前release-train版本为Ingalls-SR6*



## Repository

Spring Data预定义的Repository接口如下：

![spring-data-Repository](/images/spring-data-Repository.png)

不同的持久化方案有不同的扩展，比如：JPA、Redis、MongoDB、Neo4j、Gemfire、LDAP、Cassandra、Solr等。

对于继承`Repository`或其`子接口`的`接口`，称为`Repository接口`，Spring Data会扫描（`@Enable${store}Repositories`）且自动生成`Repository代理实现`和`bean定义`，减少了样板代码，提升了开发效率，同时又支持自定义，保持了灵活性。如果希望`Repository接口`不被自动实例化，仅作为模板接口，可以在其上使用`@NoRepositoryBean`。

### 扩展Repository

#### 自定义查询方法

##### 策略

CREATE

USE_DECLARED_QUERY

CREATE_IF_NOT_FOUND (default)

##### 创建Query

自动实现

`<查询动词><主题>by<条件>OrderBy<排序字段>Asc/Desc`

查询动词：get/read/query/find/count

主题：可省略，一般是由泛型参数化决定的，Distinct则不可省略

条件：可以使用And、Or、Between、LessThan、GreaterThan或Like连接实体属性（包括嵌套属性）来定义，`IgnoreCase`，`AllIgnoreCase`

例如：

```java
List<Person>findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);
List<Person>findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
List<Person>findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);
// Enabling ignoring case for an individual property
List<Person>findByLastnameIgnoreCase(String lastname);
// Enabling ignoring case for all suitable properties
List<Person>findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);
// Enabling static ORDER BY for a query
List<Person>findByLastnameOrderByFirstnameAsc(String lastname);
List<Person>findByLastnameOrderByFirstnameDesc(String lastname);
```

delete/remove

##### 条件

```java
List<Person>findByAddressZipCode(ZipCode zipCode);
List<Person>findByAddress_ZipCode(ZipCode zipCode);
```

##### Pageable Slice and Sort

```java
Page<User>findByLastname(String lastname, Pageable pageable);
Slice<User>findByLastname(String lastname, Pageable pageable);
List<User>findByLastname(String lastname, Sort sort);
List<User>findByLastname(String lastname, Pageable pageable);
```

##### first和top

##### Stream

##### Async

#### 声明式自定义查询

自动实现

@Query

例如：

```java
// mongoDB
@Query("{'customer':'Chuck Wagon','type':?0}")
List<Order> findChucksOrders(String type);
```

#### 混合自定义

自定义接口，需要自己实现

实现类后缀配置，默认值是`Impl`

`repositoryImplementationPostfix="Impl"`

`<SpringDataJpaRepository接口>Impl`

```java
@PersistenceContext
private EntityManager em;
```

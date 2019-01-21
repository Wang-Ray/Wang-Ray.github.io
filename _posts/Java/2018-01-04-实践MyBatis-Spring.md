---
layout: post
title: "实践MyBatis-Spring"
date: 2018-01-04 10:40:00 +0800
categories: Java
tags: database MyBatis MyBatis-Spring JDBC ORM
---

[MyBatis-Spring](http://www.mybatis.org/spring/)

## SqlSessionFactoryBean

```xml
<bean id = "sqlSessionFactory" class = "org.mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
</bean>

```

![img](/images/SqlSessionFactoryBean.png)

mapperLocations：支持通配符，比如：`classpath*:sqlmap/*-mapper.xml"`

**一般情况下，mybatis的配置文件可以不需要（mybatis-config.xml），如下情况可能需要（configLocation）：**

1. 需要定制`<settings>`，其实`configurationProperties`属性可以支持
2. 需要定制`<typeAliases>`，其实`typeHandlersPackage`属性可以支持
3. mapper xml和mapper class不在相同的classpath下，其实`mapperLocation`属性可以支持

mybatis配置文件中<environments>会被忽略。

## SqlSessionTemplate

SqlSession的一种实现，使用了代理模式，线程安全，由于每次都获取新的SQLSession，因而不支持一级缓存

```java
public class UserDaoImpl implements UserDao {
   private SqlSession sqlSession;
   public void setSqlSession(SqlSession sqlSession) {
     	this.sqlSession = sqlSession;
   }
   public User getUser(String userId) {
     	return (User) sqlSession.selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);
   }
}
```



```xml
<bean id = "sqlSession" class = "org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>

<bean id = "userDao" class = "org.mybatis.spring.sample.dao.UserDaoImpl">
  <property name="sqlSession" ref="sqlSession" />
</bean>
```



## SqlSessionDaoSupport

```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao {
	public User getUser (String userId) {
		return (User) getSqlSession().selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser",userId);
    }
}
```



```xml
<bean id="userMapper" class="org.mybatis.spring.sample.mapper.UserDaoImpl">
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```



## MapperFactoryBean

Spring中定义Mapper，线程安全，每次都获取新的SQLSession

```xml
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
	<property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper"/>
  	<property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

默认添加了依赖注入注解`@Autowired`

```java
public class MapperFactoryBean<T> extends SqlSessionDaoSupport implements FactoryBean<T>
```

```java
public abstract class SqlSessionDaoSupport extends DaoSupport {

  private SqlSession sqlSession;

  private boolean externalSqlSession;

  @Autowired(required = false)
  public final void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
    if (!this.externalSqlSession) {
      this.sqlSession = new SqlSessionTemplate(sqlSessionFactory);
    }
  }

  @Autowired(required = false)
  public final void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
    this.sqlSession = sqlSessionTemplate;
    this.externalSqlSession = true;
  }
  ...
}
```

## Mapper Scan

基于扫描自动生产Mapper

`base-package`：扫描Mapper的基础包，支持Ant匹配

`factory-ref` or `template-ref`：

`annotation`：被此注解的mapper接口，比如@Mapper

`marker-interface`：实现此接口的mapper接口，比如MyMapper

**NOTE**：`<context:component-scan/>`  won't be able to scan and register mappers. Mappers are interfaces and, in order to register them to Spring, the scanner must know how to create a `MapperFactoryBean` for each interface it finds. 

### `<mybatis:scan/>`

类似`<context:component-scan/>`

```xml
<mybatis:scan base-package="org.mybatis.spring.sample.mapper" />
```

### @MapperScan

```java
@Configuration
@MapperScan("org.mybatis.spring.sample.mapper")
public class AppConfig {

  @Bean
  public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder().addScript("schema.sql").build()
  }

  @Bean
  public SqlSessionFactory sqlSessionFactory() throws Exception {
    SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
    sessionFactory.setDataSource(dataSource());
    return sessionFactory.getObject();
  }
}
```

### MapperScannerConfigurer

推荐使用`sqlSessionFactoryBean`和`sqlSessionTemplateBean`，如果没有显示注入，由于开启了按类型自动注入，则会自动注入，参见如下源代码：

`org.mybatis.spring.mapper.ClassPathMapperScanner.processBeanDefinitions(Set<BeanDefinitionHolder>)`

```java
      if (!explicitFactoryUsed) {
        if (logger.isDebugEnabled()) {
          logger.debug("Enabling autowire by type for MapperFactoryBean with name '" + holder.getBeanName() + "'.");
        }
        definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);
      }
```
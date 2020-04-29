---
layout: post
categories: Java
tags: Java Spring ImportBeanDefinitionRegistrar
---



## 概述

`ImportBeanDefinitionRegistrar`接口主要用来注册beanDefinition，然后实例化为bean，最后注册到Spring容器中。很多三方框架集成Spring 的时候，都会通过该接口，实现扫描指定的类，然后注册到spring 容器中。
比如 Mybatis 中的Mapper接口，springCloud中的 FeignClient 接口，都是通过该接口实现的自定义注册逻辑（MyBatis对应的是MapperScannerRegistrar）。

### 接口

```java
package org.springframework.context.annotation;

import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.core.type.AnnotationMetadata;

/**
 * Interface to be implemented by types that register additional bean definitions when
 * processing @{@link Configuration} classes. Useful when operating at the bean definition
 * level (as opposed to {@code @Bean} method/instance level) is desired or necessary.
 *
 * <p>Along with {@code @Configuration} and {@link ImportSelector}, classes of this type
 * may be provided to the @{@link Import} annotation (or may also be returned from an
 * {@code ImportSelector}).
 *
 * <p>An {@link ImportBeanDefinitionRegistrar} may implement any of the following
 * {@link org.springframework.beans.factory.Aware Aware} interfaces, and their respective
 * methods will be called prior to {@link #registerBeanDefinitions}:
 * <ul>
 * <li>{@link org.springframework.context.EnvironmentAware EnvironmentAware}</li>
 * <li>{@link org.springframework.beans.factory.BeanFactoryAware BeanFactoryAware}
 * <li>{@link org.springframework.beans.factory.BeanClassLoaderAware BeanClassLoaderAware}
 * <li>{@link org.springframework.context.ResourceLoaderAware ResourceLoaderAware}
 * </ul>
 *
 * <p>See implementations and associated unit tests for usage examples.
 *
 * @author Chris Beams
 * @since 3.1
 * @see Import
 * @see ImportSelector
 * @see Configuration
 */
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}

```

## 原理

### 注册所有的`ImportBeanDefinitionRegistrar`实现类

ConfigurationClassParser.collectImports()

ConfigurationClassParser.processImports()

### 执行所有的`ImportBeanDefinitionRegistrar`的逻辑

ConfigurationClassBeanDefinitionReader.loadBeanDefinitionsForConfigurationClass()

### 自定义扫描注册`BeanDefinition`组件

## 参考

[ImportBeanDefinitionRegistrar 源码分析 （十）](https://blog.csdn.net/woshilijiuyi/article/details/85268659)
---
layout: post
categories: Java
tags: Java Spring ImportBeanDefinitionRegistrar
---
Spring Boot 2.1.6.RELEASE

SpringApplication.java

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    // 设置应用程序类型，比如Servlet、Reactive或None
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
    // 基于spring.factories设置ApplicationContextInitializer
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    // 基于spring.factories设置ApplicationListener
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

SpringApplication.java

```java
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
    // Spring应用运行监听器（SpringApplicationRunListener、ApplicationListener、SimpleApplicationEventMulticaster）
		SpringApplicationRunListeners listeners = getRunListeners(args);
    // 发布ApplicationStartingEvent
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            // 准备环境，包括系统属性，系统环境以及配置文件等
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
            // 创建容器
			context = createApplicationContext();
            // 基于spring.factories设置SpringBootExceptionReporter
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
            // 准备容器
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            // 刷新容器
			refreshContext(context);
            // 刷新容器后扩展
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```



### prepareEnvironment

准备环境

```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments) {
		// Create and configure the environment
		ConfigurableEnvironment environment = getOrCreateEnvironment();
		configureEnvironment(environment, applicationArguments.getSourceArgs());
     // 发布ApplicationEnvironmentPreparedEvent
		listeners.environmentPrepared(environment);
		bindToSpringApplication(environment);
		if (!this.isCustomEnvironment) {
			environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
					deduceEnvironmentClass());
		}
		ConfigurationPropertySources.attach(environment);
		return environment;
	}
```



ConfigFileApplicationListener

### createApplicationContext

创建容器，不同类型的应用对应不同的ApplicationContext。



None：AnnotationConfigApplicationContext
Servlet：AnnotationConfigServletWebServerApplicationContext
Reactive：AnnotationConfigReactiveWebServerApplicationContext

### prepareContext

准备容器

```java
private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment,
			SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
		context.setEnvironment(environment);
		postProcessApplicationContext(context);
		applyInitializers(context);
    // 发布ApplicationContextInitializedEvent
		listeners.contextPrepared(context);
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// Add boot specific singleton beans
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		if (printedBanner != null) {
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		// Load the sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
    // 加载启动类
		load(context, sources.toArray(new Object[0]));
    // 发布ApplicationPreparedEvent
		listeners.contextLoaded(context);
	}
```

### refreshContext

接下来的工作交给Spring，IOC和AOP等

### afterRefresh

## 参考

[ImportBeanDefinitionRegistrar 源码分析 （十）](https://blog.csdn.net/woshilijiuyi/article/details/85268659)
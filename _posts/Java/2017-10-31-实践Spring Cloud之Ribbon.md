---
layout: post
title: "实践Spring Cloud之Ribbon"
date: 2017-10-31 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Eureka
---

[Spring Cloud Ribbon](http://cloud.spring.io/spring-cloud-static/Dalston.SR4/multi/multi_spring-cloud-ribbon.html)，基于[Ribbon](https://github.com/Netflix/ribbon)实现的客户端负载均衡解决方案，这里的客户端指的是远程服务。

## 基本使用

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
```


自动基于服务名组装服务地址

```java
@EnableDiscoveryClient
@SpringBootApplication
public class HelloClientApplication {

	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(HelloClientApplication.class, args);
	}
}
```



```java
@RestController
public class HelloClientController {
	
	private final Logger logger = LoggerFactory.getLogger(getClass());
	
	@Autowired
	private RestTemplate restTemplate;

	@RequestMapping(value = "/helloClient", method = RequestMethod.GET)
	public String hello() {
		logger.info("helloClient received.");
		return restTemplate.getForEntity("http://HELLO-SERVICE/hello", String.class).getBody();
	}
}
```

也可以根据服务名查找选择服务，然后拼装服务地址。

```java
@RestController
public class HelloClientController {
	
	private final Logger logger = LoggerFactory.getLogger(getClass());
	
	@Autowired
    LoadBalancerClient loadBalancerClient;

	@Autowired
	private RestTemplate restTemplate;

	@RequestMapping(value = "/helloClient", method = RequestMethod.GET)
	public String hello() {
		logger.info("helloClient received.");
		ServiceInstance serviceInstance = loadBalancerClient.choose("hello-service");
         String url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort() + "/hello";
         return restTemplate.getForObject(url, String.class);
	}
}
```

**注：spring-cloud-starter-eureka和spring-cloud-starter-consul-discovery已经依赖了spring-cloud-starter-ribbon**



```properties
# 缓存刷新时间间隔
ribbon.ServerListRefreshInterval=30
```



## 方法

### get

### post

### put

### delete

## 原理

![ribbon-LoadBalancerClient](/images/ribbon-LoadBalancerClient.png)

* ILoadBalancer

  负载均衡器

  ![Ribbon-ILoadBalancer](/images/ribbon-ILoadBalancer.png)

  * BaseLoadBalancer
    * IPing

      检查服务实例是否正常

    * IPingStrategy

      服务检查策略

      * SerialPingStrategy

        线性遍历策略

    * IRule

      负载均衡策略

      ![ribbon-IRule](/images/ribbon-IRule.png)

      * RoundRobinRule
      * ZoneAvoidanceRule

  * DynamicServerListLoadBalancer：动态更新服务实例清单（支持和从注册中心获取）并过滤

    * ServerList

      服务清单

      ![ribbon-ServerList](/images/ribbon-ServerList.png)

      * DiscoveryEnabledNIWSServerList

    * ServerListUpdater

      服务清单更新

      * UpdateAction

    * ServerListFilter

      服务清单过滤

      ![ribbon-ServerListFilter](/images/ribbon-ServerListFilter.png)

      * ZonePreferenceServerListFilter

        ```properties
        # 基于zone优先
        eureka.instance.metadata-map.zone=zone1
        ```

        ​

  * ZoneAwareLoadBalancer：默认


## 配置

默认配置类`RibbonClientConfiguration`

```java
@SuppressWarnings("deprecation")
@Configuration
@EnableConfigurationProperties
public class RibbonClientConfiguration {

	@Value("${ribbon.client.name}")
	private String name = "client";

	// TODO: maybe re-instate autowired load balancers: identified by name they could be
	// associated with ribbon clients

    // 实现配置文件中自定义（服务级）
	@Autowired
	private PropertiesFactory propertiesFactory;

	@Bean
	@ConditionalOnMissingBean
	public IClientConfig ribbonClientConfig() {
		DefaultClientConfigImpl config = new DefaultClientConfigImpl();
		config.loadProperties(this.name);
		return config;
	}

	@Bean
	@ConditionalOnMissingBean
	public IRule ribbonRule(IClientConfig config) {
		if (this.propertiesFactory.isSet(IRule.class, name)) {
			return this.propertiesFactory.get(IRule.class, config, name);
		}
		ZoneAvoidanceRule rule = new ZoneAvoidanceRule();
		rule.initWithNiwsConfig(config);
		return rule;
	}

	@Bean
	@ConditionalOnMissingBean
	public IPing ribbonPing(IClientConfig config) {
		if (this.propertiesFactory.isSet(IPing.class, name)) {
			return this.propertiesFactory.get(IPing.class, config, name);
		}
		return new DummyPing();
	}

	@Bean
	@ConditionalOnMissingBean
	@SuppressWarnings("unchecked")
	public ServerList<Server> ribbonServerList(IClientConfig config) {
		if (this.propertiesFactory.isSet(ServerList.class, name)) {
			return this.propertiesFactory.get(ServerList.class, config, name);
		}
		ConfigurationBasedServerList serverList = new ConfigurationBasedServerList();
		serverList.initWithNiwsConfig(config);
		return serverList;
	}
	//...
```
Spring Cloud Ribbon默认实现（没有集成Eureka时） 
| BeanType                 | beanName                | ClassName                      | ClassName                                | 备注   |
| ------------------------ | ----------------------- | ------------------------------ | ---------------------------------------- | ---- |
| IClientConfig            | ribbonClientConfig      | DefaultClientConfigImpl        |                                          |      |
| IRule                    | ribbonRule              | ZoneAvoidanceRule              |                                          |      |
| IPing                    | ribbonPing              | NoOpPing                       | NIWSDiscoveryPing                        |      |
| ServerList<Server>       | ribbonServerList        | ConfigurationBasedServerList   | DiscoveryEnabledNIWSServerList,DomainExtractingServerList |      |
| ServerListFilter<Server> | ribbonServerListFilter  | ZonePreferenceServerListFilter |                                          |      |
| ILoadBalancer            | ribbonLoadBalancer      | ZoneAwareLoadBalancer          |                                          |      |
| ServerListUpdater        | ribbonServerListUpdater | PollingServerListUpdater       |                                          |      |



### 自定义配置

```java
@Configuration
public class CustomizedRibbonConfiguration{
    @Bean
    public IRule ribbonRule() {
        if (StringUtils.isEmpty(name)) {
            return null;
        }

        // 支持配置文件中按服务自定义
        if (this.propertiesFactory.isSet(IRule.class, name)) {
            return this.propertiesFactory.get(IRule.class, config, name);
        }

        LabelAndWeightMetadataRule rule = new LabelAndWeightMetadataRule();
        rule.initWithNiwsConfig(config);
        return rule;
    }
}
```
#### 自定义全局ribbon client

```java
@Configuration
@RibbonClients(defaultConfiguration = CustomizedRibbonConfiguration.class)
public class AppConfiguration {
}
```
#### 自定义服务级ribbon client

```java
@Configuration
@RibbonClient(name="hello-service" configuration = CustomizedRibbonConfiguration.class)
public class AppConfiguration {
}
```

**备注：CustomizedRibbonConfiguration不要被`@ComponentScan`扫描到，否则是全局共享的**

### 配置文件

#### 属性

ribbon相关属性定义参见`CommonClientConfigKey`

```properties
# 全局配置
ribbon.<key>=<value>
```

```properties
# 服务级配置
<clientName>.ribbon.<key>=<value>
```

例如：

```yaml
# 自定义客户地址（不通过Eureka）
hello-service:
  ribbon:
    listOfServers: example.com,google.com
# 启动加载（默认延时加载）    
ribbon:
  eager-load:
    enabled: true
    clients: hello-service, user-service
```

例如：

```properties
# 不使用eureka
ribbon.eureka.enabled = false
```

#### 服务级ribbon client

1.2.0开始支持，目前支持如下key：
```properties
NFLoadBalancerRuleClassName=
NFLoadBalancerPingClassName=
NIWSServerListClassName=
NIWSServerListFilterClassName=
NFLoadBalancerClassName=
```

例如：

```properties
hello-service.ribbon.NFLoadBalancerRuleClassName=com.github.charlesvhe.springcloud.practice.core.LabelAndWeightMetadataRule
```

**备注：不支持配置全局ribbon client，除非扩展`PropertiesFactory`**




---
layout: post
title: "实践Spring Cloud之Feign"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Feign
---

Feign实现声明式服务调用，将REST服务包装成接口，效果类似本地调用一样

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```



```java
@EnableFeignClients
```



接口

```java
/**
 * @author angi 
 */
@FeignClient("hello-service")
public interface HelloService {

	/**
	 * 无参
	 * 
	 * @return 
	 */
	@RequestMapping("/hello")
	String hello();

	@RequestMapping(value = "/hello1", method = RequestMethod.GET)
	String hello(@RequestParam("name") String name);

	/**
	 * 查询用户
	 * 
	 * @param name
	 * @param age
	 * @return
	 */
	@RequestMapping(value = "/hello2", method = RequestMethod.GET)
	User hello(@RequestHeader("name") String name, @RequestHeader("age") Integer age);

	/**
	 * 新增用户
	 * 
	 * @param user
	 * @return
	 */
	@RequestMapping(value = "/hello3", method = RequestMethod.POST)
	String hello(@RequestBody User user);
}
```



服务端

```java
/**
 * @author angi
 */
@RefreshScope
@RestController
public class HelloController {

    private final Logger logger = Logger.getLogger(getClass());

    @Value("${key2}")
    private String key;

    @Autowired
    private DiscoveryClient discoveryClient;

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String hello() throws Throwable {
        ServiceInstance instance = discoveryClient.getLocalServiceInstance();
        int sleepTime = new Random().nextInt(3000);
        Thread.sleep(sleepTime);
        logger.info("/hello, host:" + instance.getHost() + ", serviceId:" + instance.getServiceId());
        return "Hello, World!" + key;
    }

    @RequestMapping(value = "/hello1", method = RequestMethod.GET)
    public String hello(@RequestParam String name) {
        return "Hello, " + name;
    }

    /**
     * 查询用户
     *
     * @param name
     * @param age
     * @return
     */
    @RequestMapping(value = "/hello2", method = RequestMethod.GET)
    public User hello(@RequestHeader String name, @RequestHeader Integer age) {
        return new User(name, age);
    }

    /**
     * 新增用户
     *
     * @param user
     * @return
     */
    @RequestMapping(value = "/hello3", method = RequestMethod.POST)
    public String hello(@RequestBody User user) {
        return "Hello, " + user.getName() + ", " + user.getAge();
    }

}
```

feign默认使用断路器（hystrix）包裹所有方法，可以全局禁用，也可以局部禁用。

```properties
# 全局禁用hystrix
feign.hystrix.enabled=false
```


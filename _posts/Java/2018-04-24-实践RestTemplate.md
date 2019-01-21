---
layout: post
title: "实践RestTemplate"
date: 2018-04-24 11:08:00 +0800
categories: Java
tags: java RestTemplate
---

Spring 3提供的一个同步HTTP客户端，支持RESTful，默认使用JDK作为底层连接支持，也可以使用Netty、Apache HttpComponent和OkHttp等HTTP库



## TestRestTemplate

Spring Boot Test 1.4.0新增，用于支持集成测试，配合@SpringBootTest则会自动创建并注入。

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, World!";
    }
}
```



```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {ReadingListApplication.class}, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class DemoApplicationTests {

    @Autowired
    private TestRestTemplate testRestTemplate;

    @Test
    public void testController() {
        String content = testRestTemplate.getForObject("/hello", String.class);
        // 使用断言测试，使用正确的断言
        Assert.assertEquals("Hello, World!", content);
    }
}
```


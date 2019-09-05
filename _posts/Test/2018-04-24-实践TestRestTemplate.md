---
layout: post
title: "实践TestRestTemplate"
date: 2018-04-24 11:08:00 +0800
categories: 测试
tags: Test TestRestTemplate
---

Spring Boot Test 1.4.0新增，用于支持单元测试和集成测试，配合@SpringBootTest则会自动创建并注入。

代码样例

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, World!";
    }
    
    @PostMapping("/test")
    public ApiResult queryCollarBagDetail(@RequestParam("param") String param) {
           return "Hello, " + param;
    }
}
```

测试样例

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
    
    @Test
    public void testController1() {
        String content = testRestTemplate.postForObject("/test/test?param=abc", null, String.class);
        Assert.assertEquals("{\"status\":\"OK\",\"message\":\"abc:调用成功！\",\"total\":0,\"rows\":[],\"params\":null,\"code\":2000}", content);
    }
}
```


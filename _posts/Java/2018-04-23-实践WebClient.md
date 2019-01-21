---
layout: post
title: "实践WebClient"
date: 2018-04-23 11:08:00 +0800
categories: Java
tags: java WebClient
---

WebClient是Spring 5提供的一个非阻塞和响应式的HTTP客户端API

```java
	@Test
    public void webClientTest1() throws InterruptedException {
        WebClient webClient = WebClient.create("http://localhost:8080");
        Mono<String> resp = webClient
                .get().uri("/hello")
                .retrieve()
                .bodyToMono(String.class);
        resp.subscribe(logger::info);
        logger.info("main over.");
        TimeUnit.SECONDS.sleep(15);
    }
```



## webTestClient

WebTestClient是Spring 5提供的一个用于测试web服务器的非阻塞和响应式的客户端，基于WebClient实现。

```java
private final WebTestClient webTestClient = WebTestClient.bindToServer().baseUrl("http://localhost:8080").build();

    @Test
    public void testCreateUser() throws Exception {
        final User user = new User();
        user.setId(new Random().ints().toString());
        user.setName("Test");
        user.setEmail("test@example.org");
        webTestClient.post().uri("/user")
                .contentType(MediaType.APPLICATION_JSON)
                .body(Mono.just(user), User.class)
                .exchange()
                .expectStatus().isOk()
                .expectBody().jsonPath("name").isEqualTo("Test");
    }
```

如果配合@SpringBootTest则会自动创建并注入，样例如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class WebfluxDemoApplicationTests {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    public void testHelloController1() {
        webTestClient.get().uri("/hello")
                .exchange()
                .expectStatus().isOk()
                .expectBody(String.class).isEqualTo("Welcome to reactive world ~~");
    }

}
```


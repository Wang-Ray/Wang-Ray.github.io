---
layout: post
title: "实践Spring Boot Test"
date: 2018-04-25 11:22:00 +0800
categories: 测试
tags: test spring-boot-test spring-boot
---

## @SpringBootTest

针对`Spring Boot`应用的测试支持（1.4.0版本加入），当`webEnvironment`为`MOCK`时，默认可以注入`MockMvc`，当`webEnvironment`为`RANDOM_PORT`和`DEFINED_PORT`时，默认可以注入[TestRestTemplate](/测试/2018/04/24/实践TestRestTemplate/)或[WebTestClient](/java/2018/04/23/实践WebClient/)。

### value或properties

用来定义属性，`key=value`形式，例如：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(value = {"key1=value1", "key2=value2"})
public class DemoApplicationTests
```

或

```java
@RunWith(SpringRunner.class)
@SpringBootTest(properties = {"key1=value1", "key2=value2"})
public class DemoApplicationTests
```

### classes

用来指定Configuration类用于加载ApplicationContext，作用类似于`@ContextConfiguration`例如：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {ReadingListApplication.class, SampleConfig.class})
public class DemoApplicationTests
```

### webEnvironment

#### MOCK

利用仿真Web容器，不会启动真正的Web容器，默认值。

#### RANDOM_PORT

随机端口启动嵌入式Web容器

#### DEFINED_PORT

指定的端口启动嵌入式Web容器

#### NONE

### 样例

#### TestRestTemplate

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {ReadingListApplication.class}, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class WebEnviromentNotMockTests {

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

#### WebTestClient

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

## @AutoConfigureMockMvc

开启`MockMvc`自动配置，默认可以注入`MockMvc`，可以配合@SpringBootTest使用。

会启动完整Spring容器，用于集成测试。

### 样例

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class AutoConfigureMockMvcITCase {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testHelloController() throws Throwable {
        // 模拟请求，并期望正常应答
        mockMvc.perform(MockMvcRequestBuilders.get("/hello?name=Ray")).andExpect(MockMvcResultMatchers.status().isOk());

        // 模拟请求，并期望正常应答，以及期望其返回值正确
        mockMvc.perform(MockMvcRequestBuilders.get("/hello?name=Ray")).
                andExpect(MockMvcResultMatchers.content().string("Hello, Ray"));
    }
}
```

## @WebMvcTest

专门针对`Spring MVC`的`单元测试`（参数传入待测试的`Controller`），不会加载整个Spring容器，仅加载指定bean以及web层相关组件，`@Component`，`@Service`和`@Repository`等都不会加载，默认会注入`MockMvc`，如果Controller依赖其他比如@Service，则会报错（找不到相应bean），通过`Mock`解决或Import相应的bean。结合@ContextConfiguration实例化上下文。

### 样例

```java
@RunWith(SpringRunner.class)
@WebMvcTest(HelloController.class)
public class WebMvcTestTests {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private IHelloService helloService;

    @Test
    public void testHelloController() throws Throwable {
        when(helloService.hello("Ray")).thenReturn("Hello, Ray");

        // 模拟请求，并期望正常应答
        mvc.perform(MockMvcRequestBuilders.get("/hello?name=Ray")).andExpect(MockMvcResultMatchers.status().isOk());

        // 模拟请求，并期望正常应答，以及期望其返回值正确
        mvc.perform(MockMvcRequestBuilders.get("/hello?name=Ray")).
                andExpect(MockMvcResultMatchers.content().string("Hello, Ray"));
    }
}
```

## @SpringApplicationConfiguration

已被废除

## @WebIntegrationTest

已被废除
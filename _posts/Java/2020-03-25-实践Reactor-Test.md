---
layout: post
categories: Java
tags: java Reactor Test Reactor-Test
---



```xml
<dependency>
	<groupId>io.projectreactor</groupId>
	<artifactId>reactor-test</artifactId>
	<scope>test</scope>
</dependency>
```



```java
StepVerifier.create(Flux.range(1,6))
                .expectNext(1, 2, 3, 4, 5, 6)
                .expectComplete()
                .verify();
```



## StepVerifier





## 参考


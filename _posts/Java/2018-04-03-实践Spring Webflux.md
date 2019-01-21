---
layout: post
title: "实践Spring Webflux"
date: 2018-04-03 11:08:00 +0800
categories: Java
tags: java Webflux Reactor RxJava Reactive
---



## SSE

```java
public static Mono<ServerResponse> sendTimePerSec(ServerRequest serverRequest) {
    return ok().contentType(MediaType.TEXT_EVENT_STREAM).body(
            Flux.interval(Duration.ofSeconds(1)).
                    map(l -> new SimpleDateFormat("HH:mm:ss").format(new Date())), String.class);
}
```



## 不断上送

```java
    @Test
    public void webClientTest4() {
        Flux<MyEvent> eventFlux = Flux.interval(Duration.ofSeconds(1))
                .map(l -> new MyEvent(System.currentTimeMillis(), "message-" + l)).take(5);
        WebClient webClient = WebClient.create("http://localhost:8080");
        webClient
                .post().uri("/events")
                .contentType(MediaType.APPLICATION_STREAM_JSON)
                .body(eventFlux, MyEvent.class)
                .retrieve()
                .bodyToMono(Void.class)
                .block();
    }
```


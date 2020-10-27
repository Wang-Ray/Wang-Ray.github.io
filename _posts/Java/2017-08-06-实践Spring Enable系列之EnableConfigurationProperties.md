---
layout: post
title: "实践Spring Enable系列之EnableConfigurationProperties"
date: 2017-08-06 13:41:00 +0800
categories: Java
tags: spring property
---

`@EnableConfigurationProperties` enable support for `@ConfigurationProperties` annotated beans. `@ConfigurationProperties` beans can be registered in the standard way (for example using `@Bean` methods) or, for convenience, can be specified directly on this annotation.

Configuration

```java
@Configuration
// 会创建名为serverProperties的bean
@EnableConfigurationProperties(ServerProperties.class)
```

配置类

```java
@ConfigurationProperties(prefix="my")
public class ServerProperties {
    private String name;
    private Integer port;
    private List<String> servers = new ArrayList<String>();

    public String geName(){
        return this.name;
    }

    public Integer gePort(){
        return this.port;
    }
    public List<String> getServers() {
        return this.servers;
    }
}
```


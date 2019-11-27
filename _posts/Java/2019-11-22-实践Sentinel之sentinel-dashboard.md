---
layout: post
title: "实践Sentinel之sentinel-dashboard"
date: 2019-11-22 11:08:00 +0800
categories: Java
tags: java Sentinel flow-control circuit-breaking
---



### DynamicRuleProvider

```java
public interface DynamicRuleProvider<T> {

    T getRules(String appName) throws Exception;
}
```

![DynamicRuleProvider](/images/DynamicRuleProvider.png)

### DynamicRulePublisher

```java
public interface DynamicRulePublisher<T> {

    /**
     * Publish rules to remote rule configuration center for given application name.
     *
     * @param app app name
     * @param rules list of rules to push
     * @throws Exception if some error occurs
     */
    void publish(String app, T rules) throws Exception;
}
```

![DynamicRulePublisher](/images/DynamicRulePublisher.png)
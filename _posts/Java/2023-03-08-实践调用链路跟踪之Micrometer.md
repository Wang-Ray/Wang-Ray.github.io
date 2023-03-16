---
layout: post
title: "实践调用链监控之Micrometer"
categories: Java
tags: database Micrometer tracing
---

**Vendor-neutral application observability facade**

[Micrometer](https://micrometer.io/) provides a simple facade over the instrumentation clients for the most popular observability systems, allowing you to instrument your JVM-based application code without vendor lock-in. **Think SLF4J, but for observability**.
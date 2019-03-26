---
layout: post
title: "实践Spring IO Platform"
date: 2019-03-25 11:08:00 +0800
categories: Java
tags: spring spring-boot spring-io-platform
---

[Spring IO Platform](https://spring.io/projects/platform) 

## End of Life

The Platform will reach the end of its supported life on 9 April 2019. Maintenence releases of both the Brussels and Cairo lines will continue to be published up until that time. Users of the Platform are encourage to start using Spring Boot’s dependency management directory, either by using `spring-boot-starter-parent` as their Maven project’s parent, or by importing the `spring-boot-dependencies` bom.

## Overview

Spring IO is a cohesive, versioned platform for building modern applications. It is a modular, enterprise-grade distribution that delivers a curated set of dependencies while keeping developers in full control of deploying only the parts they need. Spring IO is 100% open source, lean, and modular.

## Features

- One platform, many workloads - build web, integration, batch, reactive or big data applications
- Radically simplified development experience with Spring Boot
- Production-ready features provided out of the box
- Curated and harmonized dependencies that just work together
- Modular platform that allows developers to deploy only the parts they need
- Support for embedded runtimes, classic application server, and PaaS deployments
- Depends only on Java SE, and supports Groovy, Grails and some Java EE
- Works with your existing dependency management tools such as Maven and Gradle
- The Spring IO Platform is certified to work on JDK 7 and 8 [[1](https://spring.io/projects/platform#_footnote_1)]

------

[1](https://spring.io/projects/platform#_footnoteref_1). While the Spring IO Platform supports JDK 7 and 8, many individual Spring projects also support older JDK versions. Please refer to the [individual projects' documentation](https://spring.io/docs) for the specific minimum requirements.
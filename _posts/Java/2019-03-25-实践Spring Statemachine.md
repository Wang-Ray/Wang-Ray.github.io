---
layout: post
title: "实践Spring Statemachine"
date: 2019-03-25 11:08:00 +0800
categories: Java
tags: spring spring-statemachine statemachine
---

[Spring Statemachine](https://projects.spring.io/spring-statemachine/) is a framework for application developers to use state machine concepts with Spring applications.

Spring Statemachine aims to provide following features:

- Easy to use flat one level state machine for simple use cases.
- Hierarchical state machine structure to ease complex state configuration.
- State machine regions to provide even more complex state configurations.
- Usage of triggers, transitions, guards and actions.
- Type safe configuration adapter.
- Builder pattern for easy instantiation for use outside of Spring Application context
- Recipes for usual use cases
- Distributed state machine based on a Zookeeper
- State machine event listeners.
- UML Eclipse Papyrus modeling.
- Store machine config in a persistent storage.
- Spring IOC integration to associate beans with a state machine.

State machines are powerful because behaviour is always guaranteed to be consistent, making it relatively easy to debug. This is because operational rules are written in stone when the machine is started. The idea is that your application may exist in a finite number of states and certain predefined triggers can take your application from one state to the next. Such triggers can be based on either events or timers.

It is much easier to define high level logic outside of your application and then rely on the state machine to manage state. You can interact with the state machine by sending an event, listening for changes or simply request a current state.
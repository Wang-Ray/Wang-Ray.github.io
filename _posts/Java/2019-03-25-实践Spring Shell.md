---
layout: post
title: "实践Spring Shell"
date: 2019-03-25 11:08:00 +0800
categories: Java
tags: spring spring-boot spring-shell
---

[Spring Shell](https://projects.spring.io/spring-shell/) project provides an interactive shell that allows you to plugin your own custom commands using a Spring based programming model.

## Introduction

Users of the Spring Shell project can easily build a full featured shell ( *aka* command line) application by depending on the Spring Shell jars and adding their own commands (which come as methods on spring beans). Creating a command line application can be useful *e.g.* to interact with your project's REST API, or to work with local file content.

## Features

Spring Shell's features include

- A simple, annotation driven, [programming model](https://docs.spring.io/spring-shell/docs/current-SNAPSHOT/reference/htmlsingle/#_writing_your_own_commands) to contribute custom commands
- Use of Spring Boot auto-configuration functionality as the basis for a command plugin strategy
- Tab completion, colorization, and script execution
- Customization of command prompt, shell history file name, handling of results and errors
- [Dynamic enablement](https://docs.spring.io/spring-shell/docs/current-SNAPSHOT/reference/htmlsingle/#dynamic-command-availability) of commands based on domain specific criteria
- Integration with the [bean validation API](https://docs.spring.io/spring-shell/docs/current-SNAPSHOT/reference/htmlsingle/#_validating_command_arguments)
- Already [built-in commands](https://docs.spring.io/spring-shell/docs/current-SNAPSHOT/reference/htmlsingle/#built-in-commands), such as clear screen, gorgeous help, exit
- ASCII art Tables, with formatting, alignment, fancy borders, *etc.*
---
layout: post
title: 实践反应式编程之SmallRye Mutiny
categories: Java
tags: Java Throwable Mutiny Reactive Mutiny
---

[SmallRye Mutiny](https://smallrye.io/smallrye-mutiny/) is a reactive programming library. Wait? Another one? Yes!

Mutiny is designed after having experienced many issues with other Reactive programming libraries and having seen many developers lost in an endless sequence of *flatMap*. Mutiny takes a different approach. First, Mutiny does not provide as many operators as the other famous libraries, focusing instead on the most used operators. Furthermore, Mutiny provides a more *guided* API, which avoids having classes with hundreds of methods that cause trouble for even the smartest IDE. Finally, Mutiny has built-in converters from and to other reactive programing libraries, so you can always pivot.
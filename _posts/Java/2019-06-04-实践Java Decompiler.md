---
layout: post
title: "实践Java Decompiler"
date: 2019-06-04 11:08:00 +0800
categories: Java
tags: Java Decompiler Java-Decompiler
---

[Java Decompiler](http://java-decompiler.github.io/) Yet another fast Java decompiler

The “Java Decompiler project” aims to develop tools in order to decompile and analyze Java 5 “byte code” and the later versions.

**JD-GUI** is a standalone graphical utility that displays Java source codes of “.class” files. You can browse the reconstructed source code with the JD-GUI for instant access to methods and fields.

**JD-Eclipse** is a plug-in for the Eclipse platform. It allows you to display all the Java sources during your debugging process, even if you do not have them all.

**JD-Core** is a library that reconstructs Java source code from one or more “.class” files. JD-Core may be used to recover lost source code and explore the source of Java runtime libraries. New features of Java 5, such as annotations, generics or type “enum”, are supported. JD-GUI and JD-Eclipse include JD-Core library.

JD-Core, JD-GUI & JD-Eclipse are open source projects released under the GPLv3 License.
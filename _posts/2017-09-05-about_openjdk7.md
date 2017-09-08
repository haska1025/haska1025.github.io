---
layout: post
title:  "About openjdk"
date:   2017-09-05 16:47:53
categories: java
---
### About

#### hotspot编译过程

hotspot/make/Makefile
->hotspot/make/linux/Makefile
->hotspot/make/linux/makefiles/buildtree.make

#### java 工具

javac/javah/jstack等工具是java语言实现的逻辑，但是二进制程序是c的程序，他们共用一个c main函数，路径：
jdk/src/share/bin/main.c

程序启动的时候，加载jvm，执行java代码。

#### HotSpot SA(Serviceability Agent)

#### OOP

An “oop”, or “ordinary object pointer” in HotSpot parlance is a managed pointer to an object. 
It is normally the same size as a native machine pointer. 
A managed pointer is carefully tracked by the Java application and GC subsystem, so that storage for unused objects can be reclaimed.

一个java实例在hotspot中是通过oop来表示，是一个oopDesc结构，定义在hotspot/src/share/vm/oops/oop.hpp
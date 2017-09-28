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

#### klass

java中一个类在hotspot中用klass表示，

#### klassoop

klassoop和klass在一块连续的内存中，

```c
// returns the Klass part containing dispatching behavior
Klass* klass_part() const { return (Klass*)((address)this + klass_part_offset_in_bytes()); }
```

#### 符号

Symbol类是对类似这样"java/lang/System"的java符号的抽象，Symbol除了存储符号字符串以外，还维护一个引用计数器，供GC使用。

引用计数的详细说明可以参考hotspot/src/share/vm/oops/symbol.hpp开头的描述

vmSymbols 有一个数组_symbols保存了所有Symbol，数组索引是按照java符号字符串定义的枚举值。

symbolTable 算是一个存储Symbol对象的容器，通过hash的方式保存，利于查找。


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

#### hotspot 初始化堆栈

```c
r -Xbatch -showversion -verbose:class -XX:+Verbose Queens
-verbose:class ; 打印类加载过程
-XX:+Verbose ; 打印详细信息

// java 主线程
Thread 2 (Thread 0x7ffff7fe3700 (LWP 3997)):
#0  Threads::create_vm (args=0x7ffff7fe2e00, canTryAgain=0x7ffff7fe2db3) at /home/haska/jdk7/hotspot/src/share/vm/runtime/thread.cpp:3012
#1  0x00007ffff726a935 in JNI_CreateJavaVM (vm=0x7ffff7fe2e58, penv=0x7ffff7fe2e60, args=0x7ffff7fe2e00) at /home/haska/jdk7/hotspot/src/share/vm/prims/jni.cpp:3356
#2  0x0000000000405163 in InitializeJVM (pvm=0x7ffff7fe2e58, penv=0x7ffff7fe2e60, ifn=0x7ffff7fe2ef0) at /home/haska/jdk7/hotspot/src/share/tools/launcher/java.c:1288
#3  0x0000000000403ee8 in JavaMain (_args=0x7fffffffc760) at /home/haska/jdk7/hotspot/src/share/tools/launcher/java.c:423
#4  0x00007ffff677c184 in start_thread (arg=0x7ffff7fe3700) at pthread_create.c:312
#5  0x00007ffff64a9bed in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:111

// 主线程
Thread 1 (Thread 0x7ffff7fe4740 (LWP 3978)):
#0  0x00007ffff677d65b in pthread_join (threadid=140737354020608, thread_return=0x7fffffffc688) at pthread_join.c:92
#1  0x0000000000402ebb in ContinueInNewThread (continuation=0x403e0c <JavaMain>, stack_size=1048576, args=0x7fffffffc760)
    at /home/haska/jdk7/hotspot/src/os/posix/launcher/java_md.c:1836
#2  0x0000000000403dee in main (argc=0, argv=0x7fffffffe8e8) at /home/haska/jdk7/hotspot/src/share/tools/launcher/java.c:389
```

初始化过程：

1、主线程准备好相关环境参数，会创建一个Java主线程，线程函数是JavaMain

2、java主线程函数里面调用InitializeJVM

   ->JNI_CreateJavaVM ;来创建一个java虚拟机
   
     ->Threads::create_vm  ;里面做具体的初始化工作
	   
	   ->init_globals ;src/share/vm/runtime/init.cpp == 初始化全局模块

	     ->classLoader_init ;src/share/vm/classfile/classLoader.cpp == ClassLoader初始化
		 
		   ->ClassLoader::initialize(); src/share/vm/classfile/classLoader.cpp == 引导类的加载，初始化一些性能数据，
		   
		     ->ClassLoader::setup_bootstrap_search_path; src/share/vm/classfile/classLoader.cpp == 装载类加载路径
	     
		 ->universe2_init	 
		 
		   ->SystemDictionary::initialize

  		    ->SystemDictionary::initialize_preloaded_classes;src/share/vm/classfile/systemDictionary.cpp
			   
			   ->SystemDictionary::resolve_instance_class_or_null
			     
				 ->ClassLoader::load_classfile; src/share/vm/classfile/classLoader.cpp == 真正的加载类
				 
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


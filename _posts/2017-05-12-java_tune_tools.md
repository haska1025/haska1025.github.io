---
layout: post
title:  "JAVA tune tools"
date:   2017-05-12 16:47:53
categories: java
---

### Java performance tune tools
 
 jstack,jstat,jmap
 
1. jstack

命令格式：sudo -u tomcat6 jstack -l pid

打印java线程堆栈,tomcat6代表进程所属Username，如果当前shell登录用户和java进程启动用户不一致，会报错。

pid 是要打印堆栈的java进程id



---
layout: post
title:  "About java"
date:   2018-06-26 16:47:53
categories: java
---
### About

记录java 设置、开发的一些杂项

https://beginnersbook.com/2013/05/jvm/


#### eclipse 相关

1. 调试状态下，log4j.properties文件路径

log4j.properties 放在 src目录，classes目录都不生效。

在"run configuration"->Arguments->"VM arguments" 增加 -Dlog4j.configuration="file:/D:/work/code/zookeeper/conf/log4j.properties" 可以打印出log。

2. Description Resource Path Location Type Java compiler level does not match the version of the insta

https://blog.csdn.net/zhangdaiscott/article/details/44946751

3. Access restriction

Access restriction: The method 'Launcher.getBootstrapClassPath()' is not API (restriction on required library 'C:\Program Files\Java\jdk1.8.0_172\jre\lib\rt.jar')

Eclipse 默认把这些受访问限制的API设成了ERROR。只要把Windows->Preferences->Java->Complicer-> Errors/Warnings里面的Deprecated and restricted API中的Forbidden references(access rules)选为Warning就可以编译通过

4. eclipse 解决中文字体过小的问题，简单方便

https://blog.csdn.net/myliveisend/article/details/44751423

5. Eclipse中取消生成TODO Auto-generated method stub

https://blog.csdn.net/limenghua9112/article/details/45097723

6. Eclipse常用快捷键

https://www.cnblogs.com/mq0036/p/4995390.html

#### maven

1. 添加国内仓库镜像 http://mirrors.163.com/.help/maven.html

2. 普通Java工程转换成maven工程

https://blog.csdn.net/u012012240/article/details/52370717

settings.xml路径 %mvn_home%/conf

3. maven设置下载源码

https://blog.csdn.net/ljxbbss/article/details/78060636

4. 解决maven项目在eclipse中debug时看不到源码

https://blog.csdn.net/u011001723/article/details/46504843

#### 其他

1. What's the difference between Jetty and Netty?

https://stackoverflow.com/questions/5385407/whats-the-difference-between-jetty-and-netty

2. java-concurrency

http://tutorials.jenkov.com/java-concurrency/thread-signaling.html


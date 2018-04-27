---
layout: post
title:  "zookeeper"
date:   2018-4-27 16:47:53
categories: zookeeper
---
### ACL设置

ACL的详细介绍可以参考一些文档：

https://zookeeper.apache.org/doc/r3.1.2/zookeeperProgrammers.html

https://blog.csdn.net/wuhenzhangxing/article/details/52936040

需要注意的事项

1、给Zookeeper设置ACL的目的，大概是zookeeper多套环境公用的情况下，隔离不同环境服务器访问导致的一些问题。

2、创建节点的时候需要设置ACL。至少要提供一个super user拥有read，create，delete，write，admin权限

3、增加了ACL的node，访问的时候需要add_auth

3、对于认证scheme是digest模式，user:password:permission, 其中的password需要是sha1加密后的base64.

4、对于认证scheme是ip模式, zookeeper-3.4.5 版本，不支持id 的形式为10.x.x.x/16。在zookeeper-3.5修复了此bug。


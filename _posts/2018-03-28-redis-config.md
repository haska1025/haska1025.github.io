---
layout: post
title:  "redis"
date:   2018-3-28 16:47:53
categories: redis
---
### ubuntu PPA配置

ubuntu 14.04发现版随带的是redis-2.8版本，为了使用最新版本的redis，需要配置PPA。

如下：

sudo add-apt-repository ppa:chris-lea/redis-server

sudo apt-get update

sudo apt-get install redis-server redis-sentinel

### 配置参考

1. https://redis.io/documentation

2. https://seanmcgary.com/posts/how-to-build-a-fault-tolerant-redis-cluster-with-sentinel/

3. https://docs.sensu.io/sensu-core/1.0/reference/redis/#redis-master-slave-configuration

### 注意事项

1. ubuntu14.04 发行版自带的软件包名称叫 redis-server, 提供了init script.然而 sentinel没有提供init srcipt。

2. 官方建议的redis部署方案是用源码编译部署，这样可以部署最新版本。

3. 最好是要设置密码。具体指令就是：

masterauth d8e8fca2dc0f896fd7cb4cb0031ba249
 
requirepass d8e8fca2dc0f896fd7cb4cb0031ba249

密码可以用md5sum, sha1sum, sha256sum生成一个消息摘要。

4. sentinel.conf需要配置：

sentinel auth-pass mycluster d8e8fca2dc0f896fd7cb4cb0031ba249



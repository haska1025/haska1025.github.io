---
layout: post
title:  "The kafka controller notes"
date:   2017-01-03 15:47:53
categories: kafka
---

### Kafka controller

** 1. Elect leader for kafka cluster

每一个broker启动都会尝试在/controller路径下面创建一个临时节点，成功创建的broker则选择为leader。

kafka支持多集群部署吗？如果多个集群部署，那么leader咋选举呢? /controller下面是整个系统的吧？

** 2. 更新controller epoch

在路径/controller_epoch下，更新controller epoch, 这个值貌似是一个计数器？每次启动的时候，先获取老值，然后加1，再更新。

** 3. 在路径/admin/reassign_partitions下注册监听器PartitionsReassignedListener，这是干啥的呢？

** 4. 在路径/isr_change_notification下注册监听器IsrChangeNotificationListener，这是干啥的呢？

** 5. 在路径/admin/preferred_replica_election下注册监听器PreferredReplicaElectionListener，这是干啥的呢？

** 6. 在路径/brokers/topics下注册TopicChangeListener和DeleteTopicsListener监听器

** 7. 在路径/brokers/ids下注册BrokerChangeListener监听器

** 8. zookeeper路径结构

/isr_change_notification

/admin/delete_topics

/consumers

/cluster/id

/config/topics/topic2

/config/topics/topic3

/config/clients

/config/changes

/controller

/brokers/seqid

/brokers/ids/0

/brokers/topics/topic2/partitions/0

/brokers/topics/topic3/partitions/0

** 8.0. broker 信息

{"jmx_port":-1,"timestamp":"1483499603692","endpoints":["PLAINTEXT://ubuntu:9092"],"host":"ubuntu","version":3,"port":9092}

** 8.1. 主题的分区信息

/brokers/topics/topic2的值：

{"version":1,"partitions":{"0":[0]}}

partitions的值是所有分区，每一个分区的值，是他的replica

/brokers/topics/topic3的值：

{"version":1,"partitions":{"0":[0]}}

** 8.2. 分区状态信息

/brokers/topics/topic2/partitions/0/state

{"controller_epoch":16,"leader":0,"version":1,"leader_epoch":15,"isr":[0]}

** 8.3. cluster id 信息

{"version":"1","id":"Ideo48HQT0aZEjybJS0dIw"}

** 8.4. controller leader 信息

{"version":1,"brokerid":0,"timestamp":"1483499602894"}


---
layout: post
title:  "分布式系统学习"
date:   2016-08-04 4:51:53
categories: distributed system
---

### 开源分布式中间件
1 zookeeper

2 zeromq

3 nanomsg

4 Facebook Thrift

  由facebook开发，目前贡献给了apache

5 apache Avro

6 Apache Curator

### 基本概念
1 资源共享(Make resouces accessible)

2 透明(Transparency)

Access, Location, Migration, Relocation, Replication, Concurrency, Failure

3 公开(openness)

用标准的IDL(interface description language)描述接口

complete(completeness)

neutral(neutrality)

Interoperabilty

Portability

Easy configure

Separating policy from mechanism

4 伸缩性(Scalability)

4.1 容量/规模可扩展(size)

4.2 地理位置扩展(Geographically scalability)

4.3 系统管理可扩展(administratively scalability)

4.4 扩展性问题(scalability problem)

5 架构风格(architectural style)

5.1 Layered architectures

5.2 Object-based architectures

5.3 Data-centered architectures

5.4 Event-based architectures

publish/subscribe


### 分布式算法

#### paxos

[0] <http://paxos.systems/>

[1] <http://lamport.azurewebsites.net/pubs/paxos-simple.pdf>

[2] <https://www.microsoft.com/en-us/research/uploads/prod/2016/12/The-Part-Time-Parliament.pdf>

[3] <https://stackoverflow.com/questions/22624569/leader-election-for-paxos-based-replicated-key-value-store>

[4] <https://stackoverflow.com/questions/23798724/why-is-paxos-leader-election-not-done-using-paxos>

[5] <https://mysqlhighavailability.com/the-king-is-dead-long-live-the-king-our-homegrown-paxos-based-consensus>

[6] <https://stackoverflow.com/questions/10791825/implementation-of-paxos-algorithm>

[7] <http://www.inf.usi.ch/faculty/pedone/MScThesis/marco.pdf>

#### raft

[1] <https://raft.github.io/>

### 参考

[1] <http://www.hpcs.cs.tsukuba.ac.jp/~tatebe/lecture/h23/dsys/dsd-tutorial.html>

[2] <http://stackoverflow.com/questions/7607865/something-like-apache-zookeeper-with-no-java>

[3] <http://stackoverflow.com/questions/6047917/zookeeper-alternatives-cluster-coordination-service>

[4] <http://www.tutorialspoint.com/zookeeper/index.htm>






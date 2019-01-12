---
layout: post
title:  "p2p"
date:   2019-01-12 16:47:53
categories: p2p
---

### p2p relative

#### RFC5128

rfc5128 : 主要介绍NAT 打洞技术。

Endpoint: 带表一个主机在某种IP协议的session，比如 udp或者tcp。本质是一个tuple，<ip, port, transport type(udp or tcp)>

EIM-NAT(Endpoint indepedent mapping NAT): 大意是对于NAT设备后的某个主机, 只要所有数据包都来自相同的X:x(ip:port), 不管目的地Y:y(ip:port)是否相同，

       NAT映射后的外网IP:port都是一样的。
      
non-EIM-NAT: 大意是，来自同一个socket的数据包，如果目的ip或者port不同，那么映射后的外网IP和port会发送变化。这种NAT，分两种。

         Address-Dependent Mapping: 只要来自同一个X:x的数据包，如果目的Y(目的IP相同), 那么映射后的外网IP:port都一样。不依赖目的端口y。
         
         Address and Port-Dependent Mapping： 只要是同一个Y:y，即ip和port都相同，那么映射后的外网IP:port都一样。
        
#### rfc4787

主要介绍NAT设备的各种特性，比如 ip和port转换的一些原则，包过滤原则等。

了解清楚NAT的特性，才能去做穿越相关的工作。

#### rfc5245

主要讲述ICE (Interactive Connectivity Establishment)


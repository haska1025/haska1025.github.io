---
layout: post
title:  "TCP study"
date:   2017-02-20 4:51:53
categories: tcp
---

### flow control

two ways:

1. rate-based

2. window-based

2.1 slide window

2.2 window update

  explict signaling (The window advertisement from receiver)
  
  window update and ACK in a single packet
  
### congestion control

### ISN (Initial Sequence Number)

ISN 是随机生成的，作用如下：

1. 防止网络丢包在同一个连接的不同替身之间混传，造成错误。

比如：由于网络拥塞，导致tcp断线，但是一部分网络包还在网上。等新连接建立后(两端ip+port都相同，认为是同一个tcp连接），就连接实例的包又在新连接上传输，这样会导致tcp连接出现错误，如果前后两个连接的ISN不同，那么那些旧的包对新连接的影响会几乎没有。

2. 防止网络攻击

比如：恶意破坏行为可以在某一个tcp连接上发送数据包(这个包只要保证两对 Ip+port，窗口信息这些都是合理的），堆连接造成破坏。如果ISN很难被猜测到，那么这种破坏就一定程度可以避免。


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

### 连接建立的超时重试机制

TCP的syn重发机制，采用指数退避(exponential backoff)，及每一次重试的时间是前一次的平方。
重试的次数，一般采用5次，或者6次(ubuntu16.04 默认6次）

net.ipv4.tcp_syn_retries = 6
net.ipv4.tcp_synack_retries = 5
linux上抓包显示重试过程：

16:14:16.979206 IP 192.168.1.121.56878 > 192.168.10.181.http: Flags [S], seq 993432495, win 29200, options [mss 1460,sackOK,TS val 1427054 ecr 0,nop,wscale 7], length 0
16:14:17.977774 IP 192.168.1.121.56878 > 192.168.10.181.http: Flags [S], seq 993432495, win 29200, options [mss 1460,sackOK,TS val 1427304 ecr 0,nop,wscale 7], length 0
16:14:19.981726 IP 192.168.1.121.56878 > 192.168.10.181.http: Flags [S], seq 993432495, win 29200, options [mss 1460,sackOK,TS val 1427805 ecr 0,nop,wscale 7], length 0
16:14:23.993767 IP 192.168.1.121.56878 > 192.168.10.181.http: Flags [S], seq 993432495, win 29200, options [mss 1460,sackOK,TS val 1428808 ecr 0,nop,wscale 7], length 0
16:14:32.009767 IP 192.168.1.121.56878 > 192.168.10.181.http: Flags [S], seq 993432495, win 29200, options [mss 1460,sackOK,TS val 1430812 ecr 0,nop,wscale 7], length 0
16:14:48.025791 IP 192.168.1.121.56878 > 192.168.10.181.http: Flags [S], seq 993432495, win 29200, options [mss 1460,sackOK,TS val 1434816 ecr 0,nop,wscale 7], length 0
16:15:20.089775 IP 192.168.1.121.56878 > 192.168.10.181.http: Flags [S], seq 993432495, win 29200, options [mss 1460,sackOK,TS val 1442832 ecr 0,nop,wscale 7], length 0

### RTO

1. 经典方法

SRTT = alpha * SRTT + (1-alpha) * RTT, alpha的值一般是0.8~0.9

RTO = min(ubound, max(lbound, beta*SRTT)), beta的值一般在1.3～2.0, ubound的建议值是1分钟，lbound的建议值是1秒。

2. 标准方法

srtt <- (1-g)srtt + gM, M是RTT的实时样本

rttval <- (1-h)rttval + h(|M-srtt|)

RTO = srtt + 4(rttval)

修改以后的形式

Err = M-srtt

srtt <- srtt + g(Err)

rttval <- rttval + h(|Err|-rttval)

RTO = srtt + 4(rttval)

g 是1/8, h是1/4

3. srtt 和 rttval的初始值

srtt <- M, M是第一个RTT的样本

rttval <- (1/2)M, M是第一个RTT的样本

### 重传二异性和Karn算法

如果一个TCP Segment发生了重传，即在重传Segement发送以后，收到了此Segment的ACK，那么这是对此Segment的第一次发送ACK呢，还是对重传的ACK呢？

因为搞不清楚是哪一次，会影响RTO的计算.

Karn算法对此的解决办法是，对于重传Segment的ACK不参与RTT的估值计算。

Karn算法另一个方面是重传的指数退避策略(binary exponential backoff),即每一次重传的时间是上一次重传时间间隔的2倍，直到收到了一个对非重传包的ACK。



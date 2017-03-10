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

### TCP options



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

### RTT/RTO

1. 基本概念

1.1 RTT(Round-Trip Time): 表示发送一个数据包和接收到对此数据包反馈的时间差。

RTT和应用程序的处理，通信子网中队列缓存时间，链路传输时间三部分相关。由于网络顺心万变，RTT变化也是非常大，比如网络繁忙，路由节点拥塞，会导致RTT变长。

1.2 RTO(Retransimission timeout): 当发送端没有收到接收端的反馈(ACK)，需要设置一个超时时间，补发segment给接收端。

理论上，RTO应该和网络真实RTT相当，因为这样可以确定发送端发的包就是没有被接收方收到。

然而，网络变化很快，RTT采样(RTT Sample)也是变化很大，统计学通过计算RTT的"标准偏差"(Standard deviation),可以估算出真实的RTT。

但是标准偏差需要计算平方根，而RTT的计算又是很频繁的，为了提高效率，用"均值偏差(Mean deviation)"来代替。

1.3 R1和R2

RFC1122描述了两个TCP重传的门限值R1和R2, R1描述重传的次数？R2重传失败，断开连接的时间。

linux中R1和R2的值：net.ipv4.tcp_retries1 = 3 和 net.ipv4.tcp_retries2 = 15

windows：

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\Tcpip\Parameters

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\TCPIP6\Parameters

貌似也没发现TcpMaxDataRetransmissions

1. 经典方法

SRTT = alpha * SRTT + (1-alpha) * RTT, alpha的值一般是0.8~0.9

这种方法叫EWMA(exponential weighted moving average) 或者是 low-pass filter。

RTO = min(ubound, max(lbound, beta*SRTT)), beta的值一般在1.3～2.0, ubound的建议值是1分钟，lbound的建议值是1秒。

2. 标准方法

srtt <- (1-g)srtt + gM, M是RTT的实时样本

rttvar <- (1-h)rttvar + h(|M-srtt|)

RTO = srtt + 4(rttvar)

修改以后的形式

Err = M-srtt；这是一个样本M的均值偏差(mean deviation)

srtt <- srtt + g(Err)

rttvar <- rttvar + h(|Err|-rttvar)

RTO = srtt + 4(rttvar)

g 是1/8, h是1/4

3. srtt 和 rttval的初始值

srtt <- M, M是第一个RTT的样本

rttvar <- (1/2)M, M是第一个RTT的样本

4. linux 方法

mdev = mdev(3/4) + |m-srtt|(1/4)

mdev_max = max(mdev_max, mdev)

srtt = srtt(7/8) + m(1/8)

rttvar = mdev_max

RTO = srtt + 4(rttvar)

### 重传

只有数据包，或者SYN，FIN包会被重传，空包不会被重传。

1. 重传二异性

如果一个TCP Segment发生了重传，即在重传Segement发送以后，收到了此Segment的ACK，那么这是对此Segment的第一次发送ACK呢，还是对重传的ACK呢？

因为搞不清楚是哪一次，会影响RTO的计算.

2. Karn算法

Karn算法对此的解决办法是，对于重传Segment的ACK不参与RTT的估值计算。

Karn算法另一个方面是重传的指数退避策略(binary exponential backoff),即每一次重传的时间是上一次重传时间间隔的2倍，直到收到了一个对非重传包的ACK。

3. 超时重传(Timer-based Retransimission)

4. 快速重传

dupthresh 重传门限值，表示等几个Duplicate ACK，才认为发送的Segment已经丢了，决定重传之。

达到dupthresh门限值后，TCP不会等到重传定时器，就会立即重传。

没有SACK，重传的Segment不会超过1，如果有SACK，Sender会重传超过一个Segment，填补Receiver的hole

NewReno算法

Plain Reno算法

5. SACK

SACK是在TCP的选项里面填上Receiver接收队列中的接收的Segment，格式：begin seq(4bytes) end seq(4bytes)

SACK 一般要和TSOPT一起用，对于最多支持40字节的TCP Option，最多能用3个blocks

2bytes padding + TSOPT(2+8bytes) + SACK(2 + 3*8 bytes) + 2 bytes padding





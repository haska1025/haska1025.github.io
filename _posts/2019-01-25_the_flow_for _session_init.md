---
layout: post
title:  "The flow for session initial"
date:   2019-01-25 16:47:53
categories: webrtc
---

## Pin hole 过程

              +------------+             +--------------+
              | ice-server |             | stun server  |
              +------------+             +--------------+
                   ^    /|\                 /|\  /|\
                   |  +---\------------------+    |
                   |  |    \+-----------------+   | stun req/rsp
                   |  |<--stun req/rsp        |   | 
                   |  |                       |   |
              +----+----+  stun req/rsp  +----------+
              | offerer |<-------------->| answerer |
              +---------+                +----------+

1. offerer  create offer 成功后，会通过信令server，将offer转给另一端

1.1 offerer同时会发送StunRequestBinding message给stun server

1.2 offerer收到stun server的response for StunRequestBinding message，会解析XOR_MAPPED_ADDRESS or MAPPED_ADDRESS得到 reflex addr；
    
    创建一个reflex candidate对象，接着通过信令服务器转发给answerer。

2. answerer recv offer成功后，会根据offer生成一个answer，也是通过信令服务器转给offerer

2.1 answerer 会发送StunRequestBinding message给stun server

2.1 offerer收到stun server的response for StunRequestBinding message，会解析XOR_MAPPED_ADDRESS or MAPPED_ADDRESS得到 reflex addr；
    
    创建一个reflex candidate对象，接着通过信令服务器转发给offerer。
    
3. offerer/answerer 收到信令服务器转发的对方reflex candidate，向对方发起StunRequestBinding；
   
   对于非对称NAT，A->B 发送一个数据包，B的NAT会丢弃，但是同时在A的NAT戳了一个洞，当B->A发送数据包的时候，A就可以收到;
   
   此时A再次向B发送数据包，这样就完成了P2P穿越。

### offer/answer 报文格式

下面是通过抓包获取的报文格式


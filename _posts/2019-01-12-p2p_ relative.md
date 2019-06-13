---
layout: post
title:  "p2p"
date:   2019-01-12 16:47:53
categories: p2p
---

### 介绍

#### signaling server && stun && turn server比较分析

signaling server: 基于webrtc的应用，控制服务器

stun: 建立 peer to peer 直连用的，说白了只有在non-symetric NAT下面的host，才可以建立p2p。

turn: 是一个relay server，只有当两个主机不能建立p2p直连的时候，如果仍然需要通信，那么需要turn server。

参考：

https://stackoverflow.com/questions/23715773/is-stun-server-absolutely-necessary-for-webrtc-when-i-have-a-socket-io-based-sig

### RFC标准相关

#### RFC5128

rfc5128 : 主要介绍NAT 打洞技术。

Endpoint: 带表一个主机在某种IP协议的session，比如 udp或者tcp。本质是一个tuple，<ip, port, transport type(udp or tcp)>

EIM-NAT(Endpoint indepedent mapping NAT): 大意是对于NAT设备后的某个主机, 只要所有数据包都来自相同的X:x(ip:port), 不管目的地Y:y(ip:port)是否相同， NAT映射后的外网IP:port都是一样的。
      
non-EIM-NAT: 大意是，来自同一个socket的数据包，如果目的ip或者port不同，那么映射后的外网IP和port会发生变化。这种NAT，分两种：

- Address-Dependent Mapping: 只要来自同一个X:x的数据包，如果目的Y(目的IP相同), 那么映射后的外网IP:port都一样。不依赖目的端口y。
         
- Address and Port-Dependent Mapping： 只要是同一个Y:y，即ip和port都相同，那么映射后的外网IP:port都一样。
        
#### rfc4787

主要介绍NAT设备的各种特性，比如 ip和port转换的一些原则，包过滤原则等。

了解清楚NAT的特性，才能去做穿越相关的工作。

#### rfc5245

主要讲述ICE (Interactive Connectivity Establishment)

ICE的实现有Lite和Full两种。

前者不需要连通性检查，不需要交换canditates，所以不会收集到reflex candidates,只是收集host canditates。

后者则需要连通性检查，需要部署turn或者stun server

mediasoup是 Lite implmentation

##### candidate

candidate 格式如下：

这是通过BNF(巴克斯范式）描述的，其中的 SP 表示分隔符。

```
candidate-attribute   = "candidate" ":" foundation SP component-id SP
                           transport SP
                           priority SP
                           connection-address SP     ;from RFC 4566
                           port         ;port from RFC 4566
                           SP cand-type
                           [SP rel-addr]
                           [SP rel-port]
                           *(SP extension-att-name SP
                                extension-att-value)
```

foundation:  是一个最长为32个字符(字母，数字)的字符串。  It is an
      identifier that is equivalent for two candidates that are of the
      same type, share the same base, and come from the same STUN
      server.  foundation是用在 IC E的 Frozen 算法中。
       
component-id:  is 是一个 1-256 范围内的正整数。 用于标识媒体流的 candidate 的。  此参数初始值必须是 1 。区分不同的candidate，每次都是在上一个值的基础上加 1。  对于基于 RTP 的媒体流，真实 RTP 媒体流的 candidates component ID 应该是 1, RTCP 流的 candidates 的 component ID应该是 2.  其他类型的媒体流，必须定义相关规范来表示不同流的 component ID。

transport:  用于标识 candidate 采用的传输协议。此规范只是定义了 UDP。扩展可以用 TCP， DCCP。参考 RFC4340。
     
priority:  是一个正整数。范围是 1 ~ (2^31 - 1)。
       
connection-address: candiate 的 IP 地址，可以是 IPV4，IPV6，域名。

port: candiate的端口

cand-type:  encodes the type of candidate.  This specification
      defines the values "host", "srflx", "prflx", and "relay" for host,
      server reflexive, peer reflexive, and relayed candidates,
      respectively.  The set of candidate types is extensible for the
      future.
       
例子：

a=candidate:1 1 UDP 33554431 192.168.29.76 59338 typ host

a=candidate:2621388111 1 udp 2122260223 192.168.40.107 55429 typ host generation 0 network-id 1


##### Fingerprint/Thumbprint 

Fingerprint 或者 Thumbprint(微软的叫法), 是数字证书的唯一标识。

在一些平台上，用于在证书库里面查找证书。

通过 openssl 工具获取 fingerprint 的方法。

```
SHA-256
openssl x509 -noout -fingerprint -sha256 -inform pem -in [certificate-file.crt]
 
SHA-1
openssl x509 -noout -fingerprint -sha1 -inform pem -in [certificate-file.crt]
 
MD5
openssl x509 -noout -fingerprint -md5 -inform pem -in [certificate-file.crt]
```
通过 openssl API 获取的方法如下：
```
           Hash hash = availableHashes[i];
291         unsigned int size;
292         unsigned char fingerprint[EVP_MAX_MD_SIZE];
293         char hex_fingerprint[EVP_MAX_MD_SIZE*3+1] = {0};
294
295         switch (hash)
296         {
297             case SHA1:
298                 X509_digest(certificate, EVP_sha1(), fingerprint, &size);
299                 break;
300             case SHA224:
301                 X509_digest(certificate, EVP_sha224(), fingerprint, &size);
302                 break;
303             case SHA256:
304                 X509_digest(certificate, EVP_sha256(), fingerprint, &size);
305                 break;
306             case SHA384:
307                 X509_digest(certificate, EVP_sha384(), fingerprint, &size);
308                 break;
309             case SHA512:
310                 X509_digest(certificate, EVP_sha512(), fingerprint, &size);
311                 break;
312             default:
313                 return Error("-DTLSConnection::Initialize() | Unknown hash [%d]\n",hash);
314         }
```

#### rfc4566

主要讲述SDP(session description protocol)

#### rfc5766

主要讲述turn实现

https://tools.ietf.org/html/rfc5766

#### rfc7667

rtp拓扑结构说明，比如MCU，SFU等

https://tools.ietf.org/html/rfc7667

#### rfc6347

dtls:

https://tools.ietf.org/html/rfc6347

#### rfc5246

TLS1.2

https://tools.ietf.org/html/rfc5246

MAC: Message Authentication Code

#### rfc6083

dtls over sctp

https://www.ietf.org/rfc/rfc6083.txt

#### rfc5764

DTLS-SRTP

此协议的主要特点是: DTLS 主要是用于握手，证书，秘钥交换。数据的加密是用 SRTP 协议完成。

https://tools.ietf.org/html/rfc5764

### 工程实现

这个开源项目是一个工程实现

https://github.com/coturn/coturn

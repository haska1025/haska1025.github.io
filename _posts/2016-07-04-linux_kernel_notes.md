---
layout: post
title:  "linux kernel notes"
date:   2016-06-10 4:51:53
categories: kernel
---
## linux 内核源码阅读记录

所有记录都是参考 linux-3.19.5源码

### 1 net/socket.c

  应用层和内核是通过socket来交互的，系统调用是socketcall,在 net/socket.c里面定义。

    ->
     161 // The protocol list. Each protocol is registered in here. 
     164 static DEFINE_SPINLOCK(net_family_lock);
     165 static const struct net_proto_family __rcu *net_families[NPROTO] __read_mostly;


    -> sock_register 用于各个协议栈注册协议到socket
    -> sock_init
      -> net_sysctl_init 初始化配置参数
      -> skb_init  sk buffer初始化
      -> init_inodecache
      -> register_filesystem 将协议栈注册到文件系统

### 2  结构在 include/linux/net.h

    struct socket{
       ...
       struct sock     *sk;
       const struct proto_ops  *ops;
       ...
    };

    --sock always refers to struct socket

    struct proto_ops {}

### 3 include/net/sock.h

    struct sock{} 结构是传输层socket的抽象，sk always refers to struct sock。network layer 
    representation of sockets
    struct sock {
      /*
       * Now struct inet_timewait_sock also uses sock_common, so please just
       * don't add nothing before this first member (__sk_common) --acme
       */
      struct sock_common  __sk_common;
	  
	  // sock 接收队列
	  struct sk_buff_head sk_receive_queue;
	  // 接收队列的上限，单位是字节
	  int         sk_rcvbuf;
	  // sock 异步接收队列，启动DMA才生效？
 #ifdef CONFIG_NET_DMA
      struct sk_buff_head sk_async_wait_queue;
 #endif
	  
	  // sock 发送队列
	  struct sk_buff_head sk_write_queue;
	  // 发送队列的上限，单位是字节
	  int         sk_sndbuf;
	  // 发送队列缓存的数据，单位是字节
	  int         sk_wmem_queued;
	  // 当前发送的skb
	  struct sk_buff      *sk_send_head;
    }


    struct sock_common {
       struct proto        *skc_prot;
    }
struct proto {} 结构是 socket layer -> transport layer interface



### 4 net/ipv4/af_inet.c
  
  此文件是实现ipv4的主要逻辑。

    -> inet_init ipv4协议族的初始化工作
      -> rc = proto_register(&tcp_prot, 1); 注册tcp协议的proto结构
      -> rc = proto_register(&udp_prot, 1); 注册udp协议的proto结构
      -> rc = proto_register(&raw_prot, 1); 注册原生协议的proto结构
      -> rc = proto_register(&ping_prot, 1); 注册ping协议的proto结构
      -> (void)sock_register(&inet_family_ops); 注册ipv4协议族到socket
      -> inet_add_protocol() 将icmp,udp,tcp,igmp 的net_protocol结构加入到inet_protos数组里
      -> inet_register_protosw() 将inetsw_array数组中定义的tcp,udp,icmp,raw结构注册到inetsw列表
      -> /*
      ->  *  Set the IP module up
      ->  */
      ->
      -> ip_init();
      ->
      -> tcp_v4_init();
      ->
      -> /* Setup TCP slab cache for open requests. */
      -> tcp_init();
      ->
      -> /* Setup UDP memory threshold */
      -> udp_init();
      ->
      -> /* Add UDP-Lite (RFC 3828) */
      -> udplite4_register();
      ->
      -> ping_init();

    -> ipv4协议族 
    static const struct net_proto_family inet_family_ops = {
      .family = PF_INET,
      .create = inet_create,
      .owner  = THIS_MODULE,
    };

    -> 
     // The inetsw table contains everything that inet_create needs to build a new socket.
    static struct list_head inetsw[SOCK_MAX];
    static DEFINE_SPINLOCK(inetsw_lock);
    
    -> 记录了tcp、udp、icmp、raw
    static struct inet_protosw inetsw_array[];

    ->
    const struct proto_ops inet_stream_ops; tcp协议的ops
    const struct proto_ops inet_dgram_ops; udp协议的ops

### 5 include/net/protocol.h

struct net_protocol {} 这是一个反向接口吗？当ip layer->transport layer interface

struct inet_protosw {} /* This is used to register socket interfaces for IP protocols.  */

### 6 net/ipv4/protocol.c

    -> 保存net_protocol结构的静态数组
    const struct net_protocol __rcu *inet_protos[MAX_INET_PROTOS] __read_mostly;
    const struct net_offload __rcu *inet_offloads[MAX_INET_PROTOS] __read_mostly;

### 7 net/ipv4/tcp_ipv4.c
    -> struct proto tcp_prot
    
	-> tcp_v4_init_sock 对tcp的收发buffer、窗口、拥塞控制相关变量进行初始化
	
### 8 net/ipv4/udp.c
    -> struct proto udp_prot = {

### 9 include/net/inet_sock.h

    struct inet_sock {
        ...
        /* sk and pinet6 has to be the first two members of inet_sock */
        struct sock     sk;
        #if IS_ENABLED(CONFIG_IPV6)
            struct ipv6_pinfo   *pinet6;
        #endif

    }；
### 10 include/linux/tcp.h

    struct tcp_sock {
     /* inet_connection_sock has to be the first member of tcp_sock */
     struct inet_connection_sock inet_conn;
    }；

### 11 include/net/inet_connection_sock.h 

    struct inet_connection_sock {
        /* inet_sock has to be the first member! */
        struct inet_sock      icsk_inet;
		// 拥塞控制算法相关函数
		const struct tcp_congestion_ops *icsk_ca_ops;
    }；

### 12 net/ipv4/tcp.c
    -> tcp_init
	  -> tcp_register_congestion_control(&tcp_reno); //注册拥塞控制算法，默认是reno算法
	  
### 13 include/net/tcp.h

    /* sysctl variables for tcp */
    extern int sysctl_tcp_timestamps;
    extern int sysctl_tcp_window_scaling;
    extern int sysctl_tcp_sack;


### 13 TCP接收过程主要函数调用
    -> tcp_v4_rcv         net/ipv4/tcp_ipv4.c
      -> tcp_v4_do_rcv    net/ipv4/tcp_ipv4.c
        -> tcp_rcv_established     net/ipv4/tcp_input.c
	      -> tcp_ack               net/ipv4/tcp_input.c
		    -> tcp_ack_update_window     net/ipv4/tcp_input.c;更新窗口主要是snd_wl1, snd_wnd
	          -> tcp_may_update_window   net/ipv4/tcp_input.c;窗口更新的检测条件
			  —> tcp_fast_path_check     net/ipv4/tcp_input.c;检测是否打开快速路径 
### 14 TCP数据发送函数调用
    -> tcp_sendmsg          net/ipv4/tcp.c
      -> tcp_push           net/ipv4/tcp.c
	    -> __tcp_push_pending_frames          net/ipv4/tcp_output.c
		  -> tcp_write_xmit
          
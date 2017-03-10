---
layout: post
title:  "tcp_sock_struct"
date:   2016-07-4 4:51:53
categories: kernel
---

### tcp_sock结构分析

位置：include/linux/tcp.h

#### 基础机构

    struct tcp_sock {
    /* inet_connection_sock has to be the first member of tcp_sock */
    struct inet_connection_sock inet_conn;
    u16 tcp_header_len; /* Bytes of tcp header to send      */
    u16 gso_segs;   /* Max number of segs per GSO packet    */

    /*
     *  Header prediction flags
     *  0x5?10 << 16 + snd_wnd in net byte order
	 *  预判标志，判定是否走快速处理路径
     */
    __be32  pred_flags;
    
    //都是序列号
    /*
     *  RFC793 variables by their proper names. This means you can
     *  read the code and the spec side by side (and laugh ...)
     *  See RFC793 and RFC1122. The RFC writes these in capitals.
     */
    u32 rcv_nxt;    /* What we want to receive next     */
    u32 copied_seq; /* Head of yet unread data，未读数据开头的序列号      */
    u32 rcv_wup;    /* rcv_nxt on last window update sent，窗口更新就是发送ACK？发送ACK后就更新成rcv_nxt  */
    u32 snd_nxt;    /* Next sequence we send        */

    u32 snd_una;    /* First byte we want an ack for    */
    u32 snd_sml;    /* Last byte of the most recently transmitted small packet */
    u32 rcv_tstamp; /* timestamp of last received ACK (for keepalives) */
    u32 lsndtime;   /* timestamp of last sent data packet (for restart window) */

    u32 tsoffset;   /* timestamp offset */

    struct list_head tsq_node; /* anchor in tsq_tasklet.head list */
    unsigned long   tsq_flags;

    /* Data for direct copy to user */
    struct {
        struct sk_buff_head prequeue;
        struct task_struct  *task;
        struct msghdr       *msg;
        int         memory;
        int         len;
    } ucopy;

    u32 snd_wl1;    /* Sequence for window update       */
    u32 snd_wnd;    /* The window we expect to receive  */
    u32 max_window; /* Maximal window ever seen from peer   */
    u32 mss_cache;  /* Cached effective mss, not including SACKS */

    u32 window_clamp;   /* Maximal window to advertise      */
    u32 rcv_ssthresh;   /* Current window clamp         */

    u16 advmss;     /* Advertised MSS           */
    u8  unused;
    u8  nonagle     : 4,/* Disable Nagle algorithm?             */
        thin_lto    : 1,/* Use linear timeouts for thin streams */
        thin_dupack : 1,/* Fast retransmit on first dupack      */
        repair      : 1,
        frto        : 1;/* F-RTO (RFC5682) activated in CA_Loss */
    u8  repair_queue;
    u8  do_early_retrans:1,/* Enable RFC5827 early-retransmit  */
        syn_data:1, /* SYN includes data */
        syn_fastopen:1, /* SYN includes Fast Open option */
        syn_data_acked:1,/* data in SYN is acked by SYN-ACK */
        is_cwnd_limited:1;/* forward progress limited by snd_cwnd? */
    u32 tlp_high_seq;   /* snd_nxt at the time of TLP retransmit. */
    /* RTT measurement */
    u32 srtt_us;    /* smoothed round trip time << 3 in usecs */
    u32 mdev_us;    /* medium deviation         */
    u32 mdev_max_us;    /* maximal mdev for the last rtt period */
    u32 rttvar_us;  /* smoothed mdev_max            */
    u32 rtt_seq;    /* sequence number to update rttvar */

    u32 packets_out;    /* Packets which are "in flight"    */
    u32 retrans_out;    /* Retransmitted packets out        */
    u32 max_packets_out;  /* max packets_out in last window */
    u32 max_packets_seq;  /* right edge of max_packets_out flight */

    u16 urg_data;   /* Saved octet of OOB data and control flags */
    u8  ecn_flags;  /* ECN status bits.         */
    u8  keepalive_probes; /* num of allowed keep alive probes   */
    u32 reordering; /* Packet reordering metric.        */
    u32 snd_up;     /* Urgent pointer       */

    /*
     *  Options received (usually on last packet, some only on SYN packets).
     */
    struct tcp_options_received rx_opt;
    
    /*
     *  Slow start and congestion control (see also Nagle, and Karn & Partridge)
     */
    u32 snd_ssthresh;   /* Slow start size threshold        */
    u32 snd_cwnd;   /* Sending congestion window        */
    u32 snd_cwnd_cnt;   /* Linear increase counter      */
    u32 snd_cwnd_clamp; /* Do not allow snd_cwnd to grow above this */
    u32 snd_cwnd_used;
    u32 snd_cwnd_stamp;
    u32 prior_cwnd; /* Congestion window at start of Recovery. */
    u32 prr_delivered;  /* Number of newly delivered packets to
                 * receiver in Recovery. */
    u32 prr_out;    /* Total number of pkts sent during Recovery. */

    u32 rcv_wnd;    /* Current receiver window      */
    u32 write_seq;  /* Tail(+1) of data held in tcp send buffer */
    u32 notsent_lowat;  /* TCP_NOTSENT_LOWAT */
    u32 pushed_seq; /* Last pushed seq, required to talk to windows */
    u32 lost_out;   /* Lost packets         */
    u32 sacked_out; /* SACK'd packets           */
    u32 fackets_out;    /* FACK'd packets           */
    u32 tso_deferred;

    /* from STCP, retrans queue hinting */
    struct sk_buff* lost_skb_hint;
    struct sk_buff *retransmit_skb_hint;

    /* OOO segments go in this list. Note that socket lock must be held,
     * as we do not use sk_buff_head lock.
     */
    struct sk_buff_head out_of_order_queue;

    /* SACKs data, these 2 need to be together (see tcp_options_write) */
    struct tcp_sack_block duplicate_sack[1]; /* D-SACK block */
    struct tcp_sack_block selective_acks[4]; /* The SACKS themselves*/

    struct tcp_sack_block recv_sack_cache[4];

    struct sk_buff *highest_sack;   /* skb just after the highest
                     * skb with SACKed bit set
                     * (validity guaranteed only if
                     * sacked_out > 0)
                     */
    int     lost_cnt_hint;
    u32     retransmit_high;    /* L-bits may be on up to this seqno */

    u32 lost_retrans_low;   /* Sent seq after any rxmit (lowest) */

    u32 prior_ssthresh; /* ssthresh saved at recovery start */
    u32 high_seq;   /* snd_nxt at onset of congestion   */

    u32 retrans_stamp;  /* Timestamp of the last retransmit,
                 * also used in SYN-SENT to remember stamp of
                 * the first SYN. */
    u32 undo_marker;    /* snd_una upon a new recovery episode. */
    int undo_retrans;   /* number of undoable retransmissions. */
    u32 total_retrans;  /* Total retransmits for entire connection */

    u32 urg_seq;    /* Seq of received urgent pointer */
    unsigned int        keepalive_time;   /* time before keep alive takes place */
    unsigned int        keepalive_intvl;  /* time interval between keep alive probes */

    int         linger2;

    /* Receiver side RTT estimation */
    struct {
        u32 rtt;
        u32 seq;
        u32 time;
    } rcv_rtt_est;

    /* Receiver queue space */
    struct {
        int space;
        u32 seq;
        u32 time;
    } rcvq_space;

    /* TCP-specific MTU probe information. */
    struct {
        u32       probe_seq_start;
        u32       probe_seq_end;
    } mtu_probe;
    u32 mtu_info; /* We received an ICMP_FRAG_NEEDED / ICMPV6_PKT_TOOBIG
               * while socket was owned by user.
               */

    #ifdef CONFIG_TCP_MD5SIG
    /* TCP AF-Specific parts; only used by MD5 Signature support so far */
    const struct tcp_sock_af_ops    *af_specific;

    /* TCP MD5 Signature Option information */
    struct tcp_md5sig_info  __rcu *md5sig_info;
    #endif

    /* TCP fastopen related information */
    struct tcp_fastopen_request *fastopen_req;
    /* fastopen_rsk points to request_sock that resulted in this big
     * socket. Used to retransmit SYNACKs etc.
     */
    struct request_sock *fastopen_rsk;
    };

#### tcp选项
	
    struct tcp_options_received {
        /*  PAWS/RTTM data  */
        long    ts_recent_stamp;/* Time we stored ts_recent (for aging) */
        u32 ts_recent;  /* Time stamp to echo next      */
        u32 rcv_tsval;  /* Time stamp value                 */
        u32 rcv_tsecr;  /* Time stamp echo reply            */
        u16     saw_tstamp : 1, /* Saw TIMESTAMP on last packet     */
            tstamp_ok : 1,  /* TIMESTAMP seen on SYN packet     */
            dsack : 1,  /* D-SACK is scheduled          */
            wscale_ok : 1,  /* Wscale seen on SYN packet        */
            sack_ok : 4,    /* SACK seen on SYN packet      */
            snd_wscale : 4, /* Window scaling received from sender  */
            rcv_wscale : 4; /* Window scaling to send to receiver   */
        u8  cookie_plus:6,  /* bytes in authenticator/cookie option */
            cookie_out_never:1,
            cookie_in_always:1;
        u8  num_sacks;  /* Number of SACK blocks        */
        u16 user_mss;   /* mss requested by user in ioctl   */
        u16 mss_clamp;  /* Maximal mss, negotiated at connection setup */
    };
   
#### tcp_skb_cb
    
	此结构主要是实现TCP的每一个要发送的Segment保存的信息，
	此信息占用的内存结构是sk_buff中的cb字段。
	
    /* This is what the send packet queuing engine uses to pass
     * TCP per-packet control information to the transmission code.
     * We also store the host-order sequence numbers in here too.
     * This is 44 bytes if IPV6 is enabled.
     * If this grows please adjust skbuff.h:skbuff->cb[xxx] size appropriately.
     */
    struct tcp_skb_cb {
        union {
            struct inet_skb_parm    h4;
    #if IS_ENABLED(CONFIG_IPV6)
            struct inet6_skb_parm   h6;
    #endif
        } header;   /* For incoming frames      */
        __u32       seq;        /* Starting sequence number */
        __u32       end_seq;    /* SEQ + FIN + SYN + datalen    */
        __u32       when;       /* used to compute rtt's 保存发送的时间戳    */
        __u8        tcp_flags;  /* TCP header flags. (tcp[13])  */
        __u8        sacked;     /* State flags for SACK/FACK.   */
    #define TCPCB_SACKED_ACKED  0x01    /* SKB ACK'd by a SACK block    */
    #define TCPCB_SACKED_RETRANS    0x02    /* SKB retransmitted        */
    #define TCPCB_LOST      0x04    /* SKB is lost          */
    #define TCPCB_TAGBITS       0x07    /* All tag bits         */
        __u8        ip_dsfield; /* IPv4 tos or IPv6 dsfield */
        /* 1 byte hole */
    #define TCPCB_EVER_RETRANS  0x80    /* Ever retransmitted frame */
    #define TCPCB_RETRANS       (TCPCB_SACKED_RETRANS|TCPCB_EVER_RETRANS)
   
        __u32       ack_seq;    /* Sequence number ACK'd    */
    };
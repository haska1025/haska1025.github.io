---
layout: post
title:  "TCP SACK"
date:   2017-03-23 15:47:53
categories: tcp
---

### TCP SACK&DSACK

```c

// 此函数的主要功能就是确定当前收到的SACK是否可以确定为DSAK
static int tcp_check_dsack(struct sock *sk, const struct sk_buff *ack_skb,
               struct tcp_sack_block_wire *sp, int num_sacks,
               u32 prior_snd_una)
{
    struct tcp_sock *tp = tcp_sk(sk);
    u32 start_seq_0 = get_unaligned_be32(&sp[0].start_seq);
    u32 end_seq_0 = get_unaligned_be32(&sp[0].end_seq);
    int dup_sack = 0;

	// 如果SACK序号块起始序号比ACK序号小，那么认为是DSACK
	// 例如，sack block1:[14321,15721],ack_seq=15720
    if (before(start_seq_0, TCP_SKB_CB(ack_skb)->ack_seq)) {
        dup_sack = 1;
        tcp_dsack_seen(tp);
        NET_INC_STATS_BH(sock_net(sk), LINUX_MIB_TCPDSACKRECV);
    } else if (num_sacks > 1) {
        u32 end_seq_1 = get_unaligned_be32(&sp[1].end_seq);
        u32 start_seq_1 = get_unaligned_be32(&sp[1].start_seq);
        // 第一个条件如果不成立，并且存在第二个SACK block，那么sack1的序号要完全覆盖sack0的序号，
		// 那么也认为是dsack成立
		// 例如，sack0=[15200, 16600], sack1=[13000, 16600]
        if (!after(end_seq_0, end_seq_1) &&
            !before(start_seq_0, start_seq_1)) {
            dup_sack = 1;
            tcp_dsack_seen(tp);
            NET_INC_STATS_BH(sock_net(sk),
                    LINUX_MIB_TCPDSACKOFORECV);
        }
    }

    /* D-SACK for already forgotten data... Do dumb counting. */
    if (dup_sack && tp->undo_marker && tp->undo_retrans &&
        !after(end_seq_0, prior_snd_una) &&
        after(end_seq_0, tp->undo_marker))
        tp->undo_retrans--;

    return dup_sack;
}

// 此函数难道不是判断sack block是否合法吗？怎么还扯一堆dsak呢？
static int tcp_is_sackblock_valid(struct tcp_sock *tp, int is_dsack,
                  u32 start_seq, u32 end_seq)
{
    /* Too far in future, or reversed (interpretation is ambiguous) */
	// 无效数据检测，
	// 1) SACK block的endseq，还没有发送呢
	// 2) SACK block的start_seq >= end_seq?
    if (after(end_seq, tp->snd_nxt) || !before(start_seq, end_seq))
        return 0;

    /* Nasty start_seq wrap-around check (see comments above) */
	// 无效的脏数据，start_seq 还没发送呢
    if (!before(start_seq, tp->snd_nxt))
        return 0;

    /* In outstanding window? ...This is valid exit for D-SACKs too.
     * start_seq == snd_una is non-sensical (see comments above)
     */
	 // 没错能走到这里，start_seq > snd_una，说明是合法的
    if (after(start_seq, tp->snd_una))
        return 1;

	// 能走到这一步，说明start_seq <= snd_una,也就是说sack startseq是已经ack过的。
	// 如果此sack没有被确认认为dsack，那么认为是无效的sack。
	// 为什么这样呢？
    if (!is_dsack || !tp->undo_marker)
        return 0;

    /* ...Then it's D-SACK, and must reside below snd_una completely */
	// 嗯，能走这里说明是一个dsack(start_seq < ack_seq), 如果end_seq > snd_una
	// ---------+-------------+---------+---------------+
	//   start_seq       snd_una     ack_seq         end_seq
	// 这种认为无效的sack?
    if (after(end_seq, tp->snd_una))
        return 0;

    if (!before(start_seq, tp->undo_marker))
        return 1;

    /* Too old */
    if (!after(end_seq, tp->undo_marker))
        return 0;

    /* Undo_marker boundary crossing (overestimates a lot). Known already:
     *   start_seq < undo_marker and end_seq >= undo_marker.
     */
    return !before(start_seq, end_seq - tp->max_window);
}

static int
tcp_sacktag_write_queue(struct sock *sk, const struct sk_buff *ack_skb,
            u32 prior_snd_una)
{
    const struct inet_connection_sock *icsk = inet_csk(sk);
    struct tcp_sock *tp = tcp_sk(sk);
	// 获取SACK选项的首地址，TCP_SKB_CB(ack_skb)->sacked记录了相对于tcphdr尾部的offset？
    const unsigned char *ptr = (skb_transport_header(ack_skb) +
                    TCP_SKB_CB(ack_skb)->sacked);
	// ptr+2 是skip 2bytes的padding，tcp头是4字节对齐的
    struct tcp_sack_block_wire *sp_wire = (struct tcp_sack_block_wire *)(ptr+2);
    struct tcp_sack_block sp[TCP_NUM_SACKS];
    struct tcp_sack_block *cache;
    struct tcp_sacktag_state state;
    struct sk_buff *skb;
	// ptr[0]应该是5，表示SACK选项，ptr[1]是SACK选项的长度，单位是字节数(包括了ptr[0]和ptr[1]2个byte)
	// 直接min(TCP_NUM_SACKS, ptr[1]>> 3)岂不是更好？TCPOLEN_SACK_BASE就是2吧
    int num_sacks = min(TCP_NUM_SACKS, (ptr[1] - TCPOLEN_SACK_BASE) >> 3);
    int used_sacks;
    int found_dup_sack = 0;
    int i, j;
    int first_sack_index;

    state.flag = 0;
    state.reord = tp->packets_out;

	// 之前没有
    if (!tp->sacked_out) {
        if (WARN_ON(tp->fackets_out))
            tp->fackets_out = 0;
        tcp_highest_sack_reset(sk);
    }

    found_dup_sack = tcp_check_dsack(sk, ack_skb, sp_wire,
                     num_sacks, prior_snd_una);
    if (found_dup_sack)
        state.flag |= FLAG_DSACKING_ACK;

    /* Eliminate too old ACKs, but take into
     * account more or less fresh ones, they can
     * contain valid SACK info.
     */
    if (before(TCP_SKB_CB(ack_skb)->ack_seq, prior_snd_una - tp->max_window))
        return 0;

    if (!tp->packets_out)
        goto out;

    used_sacks = 0;
    first_sack_index = 0;
    for (i = 0; i < num_sacks; i++) {
        int dup_sack = !i && found_dup_sack;

        sp[used_sacks].start_seq = get_unaligned_be32(&sp_wire[i].start_seq);
        sp[used_sacks].end_seq = get_unaligned_be32(&sp_wire[i].end_seq);

        if (!tcp_is_sackblock_valid(tp, dup_sack,
                        sp[used_sacks].start_seq,
                        sp[used_sacks].end_seq)) {
            int mib_idx;

            if (dup_sack) {
                if (!tp->undo_marker)
                    mib_idx = LINUX_MIB_TCPDSACKIGNOREDNOUNDO;
                else
                    mib_idx = LINUX_MIB_TCPDSACKIGNOREDOLD;
            } else {
                /* Don't count olds caused by ACK reordering */
                if ((TCP_SKB_CB(ack_skb)->ack_seq != tp->snd_una) &&
                    !after(sp[used_sacks].end_seq, tp->snd_una))
                    continue;
                mib_idx = LINUX_MIB_TCPSACKDISCARD;
            }

            NET_INC_STATS_BH(sock_net(sk), mib_idx);
            if (i == 0)
                first_sack_index = -1;
            continue;
        }

        /* Ignore very old stuff early */
        if (!after(sp[used_sacks].end_seq, prior_snd_una))
            continue;

        used_sacks++;
    }

    /* order SACK blocks to allow in order walk of the retrans queue */
    for (i = used_sacks - 1; i > 0; i--) {
        for (j = 0; j < i; j++) {
            if (after(sp[j].start_seq, sp[j + 1].start_seq)) {
                swap(sp[j], sp[j + 1]);

                /* Track where the first SACK block goes to */
                  if (j == first_sack_index)
                    first_sack_index = j + 1;
            }
        }
    }

    skb = tcp_write_queue_head(sk);
    state.fack_count = 0;
    i = 0;

    if (!tp->sacked_out) {
        /* It's already past, so skip checking against it */
        cache = tp->recv_sack_cache + ARRAY_SIZE(tp->recv_sack_cache);
    } else {
        cache = tp->recv_sack_cache;
        /* Skip empty blocks in at head of the cache */
        while (tcp_sack_cache_ok(tp, cache) && !cache->start_seq &&
               !cache->end_seq)
            cache++;
    }

    while (i < used_sacks) {
        u32 start_seq = sp[i].start_seq;
        u32 end_seq = sp[i].end_seq;
        int dup_sack = (found_dup_sack && (i == first_sack_index));
        struct tcp_sack_block *next_dup = NULL;

        if (found_dup_sack && ((i + 1) == first_sack_index))
            next_dup = &sp[i + 1];

        /* Skip too early cached blocks */
        while (tcp_sack_cache_ok(tp, cache) &&
               !before(start_seq, cache->end_seq))
            cache++;

        /* Can skip some work by looking recv_sack_cache? */
        if (tcp_sack_cache_ok(tp, cache) && !dup_sack &&
            after(end_seq, cache->start_seq)) {

            /* Head todo? */
            if (before(start_seq, cache->start_seq)) {
                skb = tcp_sacktag_skip(skb, sk, &state,
                               start_seq);
                skb = tcp_sacktag_walk(skb, sk, next_dup,
                               &state,
                               start_seq,
                               cache->start_seq,
                               dup_sack);
            }

            /* Rest of the block already fully processed? */
            if (!after(end_seq, cache->end_seq))
                goto advance_sp;

            skb = tcp_maybe_skipping_dsack(skb, sk, next_dup,
                               &state,
                               cache->end_seq);

            /* ...tail remains todo... */
            if (tcp_highest_sack_seq(tp) == cache->end_seq) {
                /* ...but better entrypoint exists! */
                skb = tcp_highest_sack(sk);
                if (skb == NULL)
                    break;
                state.fack_count = tp->fackets_out;
                cache++;
                goto walk;
            }

            skb = tcp_sacktag_skip(skb, sk, &state, cache->end_seq);
            /* Check overlap against next cached too (past this one already) */
            cache++;
            continue;
        }

        if (!before(start_seq, tcp_highest_sack_seq(tp))) {
            skb = tcp_highest_sack(sk);
            if (skb == NULL)
                break;
            state.fack_count = tp->fackets_out;
        }
        skb = tcp_sacktag_skip(skb, sk, &state, start_seq);

walk:
        skb = tcp_sacktag_walk(skb, sk, next_dup, &state,
                       start_seq, end_seq, dup_sack);

advance_sp:
        /* SACK enhanced FRTO (RFC4138, Appendix B): Clearing correct
         * due to in-order walk
         */
        if (after(end_seq, tp->frto_highmark))
            state.flag &= ~FLAG_ONLY_ORIG_SACKED;

        i++;
    }

    /* Clear the head of the cache sack blocks so we can skip it next time */
    for (i = 0; i < ARRAY_SIZE(tp->recv_sack_cache) - used_sacks; i++) {
        tp->recv_sack_cache[i].start_seq = 0;
        tp->recv_sack_cache[i].end_seq = 0;
    }
    for (j = 0; j < used_sacks; j++)
        tp->recv_sack_cache[i++] = sp[j];

    tcp_mark_lost_retrans(sk);

    tcp_verify_left_out(tp);

    if ((state.reord < tp->fackets_out) &&
        ((icsk->icsk_ca_state != TCP_CA_Loss) || tp->undo_marker) &&
        (!tp->frto_highmark || after(tp->snd_una, tp->frto_highmark)))
        tcp_update_reordering(sk, tp->fackets_out - state.reord, 0);

out:

#if FASTRETRANS_DEBUG > 0
    WARN_ON((int)tp->sacked_out < 0);
    WARN_ON((int)tp->lost_out < 0);
    WARN_ON((int)tp->retrans_out < 0);
    WARN_ON((int)tcp_packets_in_flight(tp) < 0);
#endif
    return state.flag;
}
                                                          
```



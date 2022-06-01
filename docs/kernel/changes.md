# Kernel changes

## 5.6 - 2020-03-29

[Linux 5.6](https://kernelnewbies.org/Linux_5.6)

* Adds `pidfd_getfd(2)` syscall.  [Grabbing file descriptors with pidfd_getfd()](https://lwn.net/Articles/808997/)

## 5.4 - 2019-11-24

[Linux 5.4](https://kernelnewbies.org/Linux_5.4)

* `waitid()` syscall supports `P_PIDFD` flag.  [Adding the pidfd abstraction to the kernel](https://lwn.net/Articles/801319/)
* `snd_wnd` added to `TCP_INFO`.  [netdev thread](https://lore.kernel.org/netdev/20190913232332.44036-2-tph@fb.com/).  Ubuntu 20.04 and Debian 11 have it.  iperf3 shows it in [3.10](https://github.com/esnet/iperf/pull/1148).
* `SOMAXCONN` increased from 128 to 4096. [commit](http://git.kernel.org/linus/19f92a030ca6d772ab44b22ee6a01378a8cb32d4), <https://lkml.org/lkml/2019/11/13/510>

## 5.3 - 2019-09-15

[Linux 5.3](https://kernelnewbies.org/Linux_5.3)

* Adds new `pidfd_open(2)` syscall.  [New system calls: pidfd_open() and close_range()](https://lwn.net/Articles/789023/)

## 5.2 - 2019-07-07

[Linux 5.2](https://kernelnewbies.org/Linux_5.2)

* Adds the `CLONE_PIDFD` flag to `clone(2)`. [Rethinking race-free process signaling](https://lwn.net/Articles/784831/)

## 5.1 - 2019-05-05

[Linux 5.1](https://kernelnewbies.org/Linux_5.1)

* Adds `pidfd_send_signal(2)` syscall.  [Toward race-free process signaling](https://lwn.net/Articles/773459/)

## 4.17 - 2018-06-03

[Linux 4.17](https://kernelnewbies.org/Linux_4.17#Kernel_TLS_receive_path)

* Kernel TLS receive path

## 4.15 - 2018-01-28

[Linux 4.15](https://kernelnewbies.org/Linux_4.15)

* tcp: implement rb-tree based retransmit queue [commit 75c119afe14f7](http://git.kernel.org/linus/75c119afe14f74b4dd967d75ed9f57ab6c0ef045)

```diff
diff --git a/include/net/sock.h b/include/net/sock.h
index a6b9a8d1a6df..4827094f1db4 100644
--- a/include/net/sock.h
+++ b/include/net/sock.h
@@ -397,7 +397,10 @@ struct sock {
        int                     sk_wmem_queued;
        refcount_t              sk_wmem_alloc;
        unsigned long           sk_tsq_flags;
-       struct sk_buff          *sk_send_head;
+       union {
+               struct sk_buff  *sk_send_head;   // Front of stuff to transmit
+               struct rb_root  tcp_rtx_queue;   // TCP re-transmit queue
+       };
        struct sk_buff_head     sk_write_queue;  // Packet sending queue
        __s32                   sk_peek_off;
        int                     sk_write_pending;
```

* TUN: enable NAPI for TUN/TAP driver [commit 1](http://git.kernel.org/linus/943170998b200190f99d3fe7e771437e2c51f319),
    [commit 2](http://git.kernel.org/linus/90e33d45940793def6f773b2d528e9f3c84ffdc7)

## 4.13 - 2017-09-03

[Linux 4.13](https://kernelnewbies.org/Linux_4.13#Kernel_TLS_acceleration)

* [TLS in the kernel](https://lwn.net/Articles/666509/), sending only.

## 4.12 - 2017-07-02

[Linux 4.12](https://kernelnewbies.org/Linux_4.12)

* Removed `net.ipv4.tcp_tw_recycle` option from Kernel [commit](https://git.kernel.org/linus/4396e46187ca5070219b81773c4e65088dac50cc).
Ref. [Coping with the TCP TIME-WAIT state on busy Linux servers](https://vincent.bernat.ch/en/blog/2014-tcp-time-wait-state-linux) by Vincent Bernat.

## 4.10 - 2017-02-19

[Linux 4.10](https://kernelnewbies.org/Linux_4.10)

* tcp: sender chronographs instrumentation
* tcp: drop SYN packets if accept queue is full [commit](https://git.kernel.org/linus/5ea8ea2cb7f1d0db15762c9b0bb9e7330425a071).

## 4.9 - 2016-12-11

[Linux 4.9](https://kernelnewbies.org/Linux_4.9)

* BBR congestion control algorithm
* tcp: use an RB tree for ooo receive queue [commit 9f5afeae51](https://git.kernel.org/linus/9f5afeae51526b3ad7b7cb21ee8b145ce6ea7a7a)

```diff
diff --git a/include/linux/tcp.h b/include/linux/tcp.h
index 7be9b1242354..c723a465125d 100644
--- a/include/linux/tcp.h
+++ b/include/linux/tcp.h
@@ -281,10 +281,9 @@ struct tcp_sock {
        struct sk_buff* lost_skb_hint;
        struct sk_buff *retransmit_skb_hint;

-       /* OOO segments go in this list. Note that socket lock must be held,
-        * as we do not use sk_buff_head lock.
-        */
-       struct sk_buff_head     out_of_order_queue;
+       /* OOO segments go in this rbtree. Socket lock must be held. */
+       struct rb_root  out_of_order_queue;
+       struct sk_buff  *ooo_last_skb; /* cache rb_last(out_of_order_queue) */
```

## 4.8 - 2016-10-02

[Linux 4.8](https://kernelnewbies.org/Linux_4.8)

* [Reinventing the timer wheel](https://lwn.net/Articles/646950/) [merge](https://git.kernel.org/torvalds/c/55392c4c06204c8149dc333309cf474691f1cc3c)

## 4.6 - 2016-05-15

[Linux 4.6](https://kernelnewbies.org/Linux_4.6)

* Faster SO_REUSEPORT for TCP

## 4.5 - 2016-03-13

[Linux 4.5](https://kernelnewbies.org/Linux_4.5)

* Faster SO_REUSEPORT for UDP
* [Better epoll multithread scalability](https://lwn.net/Articles/633422)

## 4.4 - 2016-01-10

[Linux 4.4](https://kernelnewbies.org/Linux_4.4)

* TCP scalability
    - Lockless TCP listener
    - SO_INCOMING_CPU
    - TCP_NEW_SYN_RECV

```text
commit ca6fb06518836ef9b65dc0aac02ff97704d52a05
Author: Eric Dumazet <edumazet@google.com>
Date:   Fri Oct 2 11:43:35 2015 -0700

    tcp: attach SYNACK messages to request sockets instead of listener

    If a listen backlog is very big (to avoid syncookies), then
    the listener sk->sk_wmem_alloc is the main source of false
    sharing, as we need to touch it twice per SYNACK re-transmit
    and TX completion.

    (One SYN packet takes listener lock once, but up to 6 SYNACK
    are generated)

    By attaching the skb to the request socket, we remove this
    source of contention.

    Tested:

     listen(fd, 10485760); // single listener (no SO_REUSEPORT)
     16 RX/TX queue NIC
     Sustain a SYNFLOOD attack of ~320,000 SYN per second,
     Sending ~1,400,000 SYNACK per second.
     Perf profiles now show listener spinlock being next bottleneck.

        20.29%  [kernel]  [k] queued_spin_lock_slowpath
        10.06%  [kernel]  [k] __inet_lookup_established
         5.12%  [kernel]  [k] reqsk_timer_handler
         3.22%  [kernel]  [k] get_next_timer_interrupt
         3.00%  [kernel]  [k] tcp_make_synack
         2.77%  [kernel]  [k] ipt_do_table
         2.70%  [kernel]  [k] run_timer_softirq
         2.50%  [kernel]  [k] ip_finish_output
         2.04%  [kernel]  [k] cascade

    Signed-off-by: Eric Dumazet <edumazet@google.com>
    Signed-off-by: David S. Miller <davem@davemloft.net>
```

```text
commit 079096f103faca2dd87342cca6f23d4b34da8871
Author: Eric Dumazet <edumazet@google.com>
Date:   Fri Oct 2 11:43:32 2015 -0700

    tcp/dccp: install syn_recv requests into ehash table

    In this patch, we insert request sockets into TCP/DCCP
    regular ehash table (where ESTABLISHED and TIMEWAIT sockets
    are) instead of using the per listener hash table.

    ACK packets find SYN_RECV pseudo sockets without having
    to find and lock the listener.

    In nominal conditions, this halves pressure on listener lock.

    Note that this will allow for SO_REUSEPORT refinements,
    so that we can select a listener using cpu/numa affinities instead
    of the prior 'consistent hash', since only SYN packets will
    apply this selection logic.

    We will shrink listen_sock in the following patch to ease
    code review.

    Signed-off-by: Eric Dumazet <edumazet@google.com>
    Cc: Ying Cai <ycai@google.com>
    Cc: Willem de Bruijn <willemb@google.com>
    Signed-off-by: David S. Miller <davem@davemloft.net>
```

## 4.3 - 2015-11-01

[Linux 4.3](https://kernelnewbies.org/Linux_4.3)

* Adds direct Sockets syscalls to i386.

```text
commit 9dea5dc921b5f4045a18c63eb92e84dc274d17eb
Author: Andy Lutomirski <luto@kernel.org>
Date:   Tue Jul 14 15:24:24 2015 -0700

    x86/entry/syscalls: Wire up 32-bit direct socket calls

    On x86_64, there's no socketcall syscall; instead all of the
    socket calls are real syscalls.  For 32-bit programs, we're
    stuck offering the socketcall syscall, but it would be nice to
    expose the direct calls as well.  This will enable seccomp to
    filter socket calls (for new userspace only, but that's fine for
    some applications) and it will provide a tiny performance boost.
```

[glibc 2.23](https://www.sourceware.org/ml/libc-alpha/2016-02/msg00502.html) 2016-02-19.
Ubuntu 16.04 supports it, Debian 8 doesn't.

<https://github.com/bminor/glibc/commit/e5a5315e2d290fe34e0fb80996c713b8b802dcc9>

```text
commit e5a5315e2d290fe34e0fb80996c713b8b802dcc9
Author: Joseph Myers <joseph@codesourcery.com>
Date:   Wed Dec 9 20:59:43 2015 +0000

    Use direct socket syscalls for new kernels on i386, m68k, microblaze, sh.

    Now that we have __ASSUME_* macros for direct socket syscalls to use
    them instead of socketcall when they can be assumed to be available on
    socketcall architectures, this patch defines those macros when
    appropriate for i386, m68k, microblaze and sh (for 4.3, 4.3, all
    supported kernels and 2.6.37, respectively; the only use of socketcall
    support on microblaze is it allows accept4 and sendmmsg to be
    supported on a wider range of kernel versions).
```

## 4.2 - 2015-08-30

[Linux 4.2](https://kernelnewbies.org/Linux_4.2)

```text
commit 90c337da1524863838658078ec34241f45d8394d
Author: Eric Dumazet <edumazet@google.com>
Date:   Sat Jun 6 21:17:57 2015 -0700

    inet: add IP_BIND_ADDRESS_NO_PORT to overcome bind(0) limitations

    When an application needs to force a source IP on an active TCP socket
    it has to use bind(IP, port=x).

    As most applications do not want to deal with already used ports, x is
    often set to 0, meaning the kernel is in charge to find an available
    port.
    But kernel does not know yet if this socket is going to be a listener or
    be connected.
    It has very limited choices (no full knowledge of final 4-tuple for a
    connect())

    With limited ephemeral port range (about 32K ports), it is very easy to
    fill the space.

    This patch adds a new SOL_IP socket option, asking kernel to ignore
    the 0 port provided by application in bind(IP, port=0) and only
    remember the given IP address.

    The port will be automatically chosen at connect() time, in a way
    that allows sharing a source port as long as the 4-tuples are unique.

    This new feature is available for both IPv4 and IPv6 (Thanks Neal)
```

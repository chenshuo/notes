# 4.12 - 2017-07-02

* Removed `net.ipv4.tcp_tw_recycle` option from Kernel [commit](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=4396e46187ca5070219b81773c4e65088dac50cc).
Ref. [Coping with the TCP TIME-WAIT state on busy Linux servers](https://vincent.bernat.ch/en/blog/2014-tcp-time-wait-state-linux) by Vincent Bernat.

# 4.9 - 2016-12-11

[Linux 4.9](https://kernelnewbies.org/Linux_4.9)

* BBR congestion control algorithm
* tcp: use an RB tree for ooo receive queue [commit 9f5afeae51](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=9f5afeae51526b3ad7b7cb21ee8b145ce6ea7a7a)

# 4.6 - 2016-05-15

[Linux 4.6](https://kernelnewbies.org/Linux_4.6)

* Faster SO_REUSEPORT for TCP

# 4.5 - 2016-03-13

[Linux 4.5](https://kernelnewbies.org/Linux_4.5)

* Faster SO_REUSEPORT for UDP
* [Better epoll multithread scalability](https://lwn.net/Articles/633422)

# 4.4 - 2016-01-10

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

# 4.3 - 2015-11-01

[Linux 4.3](https://kernelnewbies.org/Linux_4.3)

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

# 4.2 - 2015-08-30

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

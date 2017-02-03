# 4.9 - 2016-12-11

[Linux 4.9](https://kernelnewbies.org/Linux_4.9)

* BBR congestion control algorithm

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

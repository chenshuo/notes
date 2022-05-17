# Study Kernel source


## Build kernel from source for debugging

Generally, Linux kernel doesn't compile with `-O0`, which is unfortunate
for debugging and steping through in GDB.

For parts I (Chen Shuo) care mostly, i.e. networking, I manage to compile
those parts with `-O0` and the rest with `-Og`, so GDB works much smoother.

I created a git repository for my own convenience.  It has some tweaks in
`kernel/configs/study.config` to make building and debugging faster.

```sh
$ cd ~/kernel
$ git clone https://github.com/chenshuo/linux-study.git
$ cd linux-study

$ study/config.sh
...
To build:
    cd ../build-g
    make -j8

$ cd ../build-g
$ make -j24
...
  BUILD   arch/x86/boot/bzImage
Kernel: arch/x86/boot/bzImage is ready
```

## Debugging QEMU and virtme

Traditionally, you need to [build a disk image with root filesystem](http://nickdesaulniers.github.io/blog/2018/10/24/booting-a-custom-linux-kernel-in-qemu-and-debugging-it-with-gdb/),
but with [virtme](https://github.com/amluto/virtme), it's not needed anymore.

Virtme shares host filesystem with guest OS using 9p protocol, so it doesn't interfere with networking stack.


```sh
$ virtme/virtme-run --kdir ~/kernel/build-g --mods=auto --pwd --qemu-opts -s
```

In another window

```sh
$ gdb vmlinux

(gdb) target remote :1234
0xffffffff818de078 in default_idle () at /home/chenshuo/kernel/linux-study/arch/x86/kernel/process.c:718
718	}

(gdb) b sock_alloc
(gdb) continue
```

To exit, in QEMU window: `Ctrl-A C` then `quit`.

## `ftrace()` to show call graph

[Steven Rostedt - Learning the Linux Kernel with tracing](https://www.youtube.com/watch?v=JRyrhsx-L5Y)

Sample function graph trace for `tcp_sendmsg()`:

```text
 0)               |  tcp_sendmsg() {
 0)               |    lock_sock_nested() {
 0)   2.032 us    |    }
 0)               |    tcp_sendmsg_locked() {
 0)   0.753 us    |      tcp_rate_check_app_limited();
 0)               |      tcp_send_mss() {
 0)               |        tcp_current_mss() {
 0)   1.826 us    |        }
 0)   0.387 us    |        tcp_xmit_size_goal();
 0)   2.858 us    |      }
 0)               |      sk_stream_alloc_skb() {
 0)   7.404 us    |      }
 0)               |      skb_entail() {
 0)   2.389 us    |      }
 0)               |      sk_page_frag_refill() {
 0)   6.727 us    |      }
 0)               |      __sk_mem_schedule() {
 0)   1.750 us    |      }
 0)               |      tcp_push_one() {
 0)               |        tcp_write_xmit() {
 0)               |          tcp_mstamp_refresh() {
 0)   0.629 us    |          }
 0)               |          tcp_tso_segs() {
 0)   0.158 us    |            tcp_tso_autosize();
 0)   0.569 us    |          }
 0)   0.498 us    |          tcp_pacing_check();
 0)               |          tcp_init_tso_segs() {
 0)   0.166 us    |            tcp_set_skb_tso_segs();
 0)   0.618 us    |          }
 0)   0.328 us    |          tcp_snd_wnd_test();
 0)   0.233 us    |          tcp_mss_split_point();
 0)               |          tso_fragment();
 0)   0.240 us    |          tcp_small_queue_check();
 0)               |          tcp_transmit_skb() {
 0)               |            __tcp_transmit_skb() {
 0)               |              skb_clone() 
 0)   0.216 us    |              tcp_established_options();
 0)   0.184 us    |              skb_push();
 0)               |              tcp_options_write() 
 0)               |              tcp_select_window() 
 0)   0.167 us    |              tcp_ecn_send();
 0)   0.148 us    |              bpf_skops_write_hdr_opt();
 0)               |              tcp_v6_send_check() 
 0)               |              tcp_event_data_sent() 
 0)               |              inet6_csk_xmit() {
 0)               |                inet6_csk_route_socket() 
 0)               |                ip6_xmit() {
 0)   0.173 us    |                  skb_push();
 0)               |                  ip6_dst_hoplimit() 
 0)   0.188 us    |                  ip6_autoflowlabel();
 0)               |                  ip6_mtu() 
 0)               |                  ip6_output() {
 0)               |                    ip6_finish_output() {
 0)               |                      __ip6_finish_output() {
 0)               |                        ip6_mtu() 
 0)               |                        skb_gso_validate_network_len() 
 0)               |                        ip6_finish_output2() {
 0)               |                          neigh_connected_output() {
 0)               |                            dev_queue_xmit() {
 0)               |                              __dev_queue_xmit() {
 0)   0.531 us    |                                qdisc_pkt_len_init();
 0)   0.227 us    |                                netdev_core_pick_tx();
 0)   0.160 us    |                                _raw_spin_lock();
 0)               |                                sch_direct_xmit() {
 0)               |                                  validate_xmit_skb_list()
 0)               |                                  dev_hard_start_xmit() {
 0)               |                                    xmit_one() {
 0)   0.197 us    |                                      dev_nit_active();
 0)               |                                      dev_queue_xmit_nit()
 0)               |                                      tun_net_xmit() {
 0)   0.165 us    |                                        tun_automq_xmit();
 0)   0.155 us    |                                        check_filter();
 0)   0.421 us    |                                        run_ebpf_filter();
 0)               |                                        tcp_wfree() {
 0)   1.247 us    |                                        }
 0)   4.366 us    |                                      }
 0) + 11.675 us   |                                    }
 0) + 12.107 us   |                                  }
 0)   0.144 us    |                                  _raw_spin_lock();
 0) + 16.900 us   |                                }
 0)               |                                __qdisc_run() {
 0)   0.255 us    |                                  dequeue_skb();
 0)   0.602 us    |                                }
 0)   0.160 us    |                                __local_bh_enable_ip();
 0) + 20.803 us   |                              }
 0) + 21.102 us   |                            }
 0) + 21.831 us   |                          }
 0)   0.180 us    |                          __local_bh_enable_ip();
 0) + 22.979 us   |                        }
 0) + 25.719 us   |                      }
 0) + 26.057 us   |                    }
 0) + 26.430 us   |                  }
 0) + 29.684 us   |                }
 0)   0.164 us    |                rcu_read_unlock_strict();
 0) + 35.114 us   |              }
 0)   0.334 us    |              tcp_update_skb_after_send();
 0)   0.302 us    |              tcp_rate_skb_sent();
 0) + 44.667 us   |            }
 0) + 45.075 us   |          }
...
```

Clearly shows the call chain:

* Transport layer

```c
tcp_sendmsg() -> tcp_sendmsg_locked() -> tcp_push_one() -> tcp_write_xmit() -> tcp_transmit_skb() -> __tcp_transmit_skb()
```

* Network layer

```c
inet6_csk_xmit() -> ip6_xmit() -> ip6_output() -> ip6_finish_output() -> __ip6_finish_output() -> ip6_finish_output2()
```

* Device layer

```c
neigh_connected_output() -> dev_queue_xmit() -> __dev_queue_xmit() -> sch_direct_xmit() -> dev_hard_start_xmit() -> xmit_one()
```

* Driver layer
```c
tun_net_xmit()
```
## Study TCP/IP stack using `packetdrill`

## Compile FreeBSD kernel on Linux

Recent FreeBSD kernel can be cross compiled on Linux host: https://wiki.freebsd.org/BuildingOnNonFreeBSD

### Debugging FreeBSD kernel in QEMU


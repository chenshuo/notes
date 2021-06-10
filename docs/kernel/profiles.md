# Kernel profiles



```shell
z420$ lscpu
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   46 bits physical, 48 bits virtual
CPU(s):                          8
On-line CPU(s) list:             0-7
Thread(s) per core:              2
Core(s) per socket:              4
Socket(s):                       1
NUMA node(s):                    1
Vendor ID:                       GenuineIntel
CPU family:                      6
Model:                           45
Model name:                      Intel(R) Xeon(R) CPU E5-1620 0 @ 3.60GHz
Stepping:                        7
CPU MHz:                         2393.980
CPU max MHz:                     3800.0000
CPU min MHz:                     1200.0000
BogoMIPS:                        7182.25
Virtualization:                  VT-x
L1d cache:                       128 KiB
L1i cache:                       128 KiB
L2 cache:                        1 MiB
L3 cache:                        10 MiB
NUMA node0 CPU(s):               0-7
Vulnerability Itlb multihit:     KVM: Mitigation: VMX disabled
Vulnerability L1tf:              Mitigation; PTE Inversion; VMX conditional cache flushes, SMT vulnerable
Vulnerability Mds:               Vulnerable: Clear CPU buffers attempted, no microcode; SMT vulnerable
Vulnerability Meltdown:          Mitigation; PTI
Vulnerability Spec store bypass: Vulnerable
Vulnerability Spectre v1:        Mitigation; usercopy/swapgs barriers and __user pointer sanitization
Vulnerability Spectre v2:        Mitigation; Full generic retpoline, STIBP disabled, RSB filling
Vulnerability Srbds:             Not affected
Vulnerability Tsx async abort:   Not affected
Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush
                                  dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_
                                 tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf
                                 pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr pdcm pcid d
                                 ca sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx lahf_lm epb pti
                                  tpr_shadow vnmi flexpriority ept vpid xsaveopt dtherm ida arat pln pts
```


## `dd if=/dev/zero of=/dev/null`

Writing to `/dev/null` is a no-op, it doesn't even touch the buffer.
That is, even if you pass a NULL pointer to `write(2)`, e.g. `write(null_fd, NULL, 1048576)`,
it won't segfault.

```c
// drivers/char/mem.c
static ssize_t write_null(struct file *file, const char __user *buf,
                          size_t count, loff_t *ppos)
{
        return count;
}
```


* [profile](profile-dd1b.html) of `sudo perf record -g dd if=/dev/zero of=/dev/null bs=1 count=10M`

```shell
$ sudo perf record -g -o bs1.data dd if=/dev/zero of=/dev/null bs=1 count=10M
10485760+0 records in
10485760+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 5.82431 s, 1.8 MB/s
[ perf record: Woken up 9 times to write data ]
[ perf record: Captured and wrote 2.153 MB bs1.data (23322 samples) ]
```

[ddbs1b10M.pb.gz](ddbs1b10M.pb.gz)

* [profile](profile-dd128k.html) of `sudo perf record -g dd if=/dev/zero of=/dev/null bs=128k count=1M`

```shell
$ sudo perf record -g -o bs128k.data dd if=/dev/zero of=/dev/null bs=128k count=1M
1048576+0 records in
1048576+0 records out
137438953472 bytes (137 GB, 128 GiB) copied, 6.35848 s, 21.6 GB/s
[ perf record: Woken up 12 times to write data ]
[ perf record: Captured and wrote 3.052 MB bs128k.data (25460 samples) ]
```

[ddbs128k1M.pb.gz](ddbs128k1M.pb.gz)

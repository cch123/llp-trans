Appendix D. Performance Tests Information

所有性能测试都是在下面机器上完成的：

```
> uname -a

Linux perseus 3.16-2-amd64 #1 SMP Debian 3.16.3-2 (2014-09-20) x86_64 GNU/Linux

> cat /proc/cpuinfo

processor           : 0
vendor_id          : GenuineIntel
cpu family :6
model: 69
model name: Intel(R) Core(TM) i5-4210U CPU @ 1.70GHz
stepping:1
microcode: 0x1d
cpu MHz: 2394.458
cache size: 3072 KB
physical id:0
siblings:1
core id:0
cpu cores :1
apicid : 0
initial apicid  : 0
fpu : yes
fpu_exception : yes
cpuid level : 13
wp : yes
flags : fpu vme de pse tsc msr pae mce cx8 apic
sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse
sse2 ss syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon
pebs bts nopl xtopology tsc_reliable nonstop_tsc aperfmperf
pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe
popcnt aes xsave avx f16c rdrand hypervisor lahf_lm ida arat
epb pln pts dtherm fsgsbase smep

bogomips        : 4788.91
clflush size    : 64
cache_alignment : 64
address sizes   : 40 bits physical, 48 bits virtual
power management:
processor: 1
vendor_id: GenuineIntel
cpu family: 6
model: 69
model name: Intel(R) Core(TM) i5-4210U CPU @ 1.70GHz
stepping: 1
microcode: 0x1d
cpu MHz: 2394.458
cache size: 3072 KB
physical id: 2
siblings: 1
core id: 0
cpu cores: 1
apicid: 2
initial apicid  : 2
fpu :  yes
fpu_exception : yes
cpuid_level  : 13
wp : yes

flags : fpu vme de pse tsc msr pae mce cx8 apic
sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse
sse2 ss syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon
pebs bts nopl xtopology tsc_reliable nonstop_tsc aperfmperf
pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe
popcnt aes xsave avx f16c rdrand hypervisor lahf_lm ida arat
epb pln pts dtherm fsgsbase smep
bogomips        : 4788.91
clflush size    : 64
cache_alignment : 64
address sizes   : 40 bits physical, 48 bits virtual
power management:

> cat /proc/meminfo
MemTotal:        1017348 kB
MemFree:          516672 kB
MemAvailable:     565600 kB
Buffers:           32756 kB
Cached:           114944 kB
SwapCached:        10044 kB
Active:           376288 kB
Inactive:          49624 kB
Active(anon):     266428 kB
Inactive(anon):    12440 kB
Active(file):     109860 kB
Inactive(file):    37184 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:        901116 kB
SwapFree:         868356 kB
Dirty:                44 kB
Writeback:             0 kB
AnonPages:        270964 kB
Mapped:            43852 kB
Shmem:               648 kB
Slab:              45980 kB
SReclaimable:      29016 kB
SUnreclaim:        16964 kB
KernelStack:        4192 kB
PageTables:         6100 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     1409788 kB
Committed_AS:    1212356 kB
VmallocTotal:   34359738367 kB
VmallocUsed:      145144 kB
VmallocChunk:   34359590172 kB
HardwareCorrupted:     0 kB
HardwareCorrupted:     0 kB
AnonHugePages:         0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:       49024 kB
DirectMap2M:      999424 kB
DirectMap1G:           0 kB

```




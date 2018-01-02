16.2.4 越过缓存

也存在一种越过缓存而直接向内存写入数据的方法，即使在用户模式下也可支持，为了使用这种方法，我们应该有对页表项的访问权限，因为该项中存储了 CWT 位信息。在 Intel 64 架构，movntps 这条指令可以让我们来完成这件事情。操作系统自身会在内存映射 IO 发生时设置 CWT 这一位\(cache-write-through\) 到页表，这种情况下虚拟内存被用作了外部设备的一个访问接口。

在该场景下，任何内存的读写都会导致缓存失效，对性能的影响比较大，无法带来任何好处。

GCC 有一些内置方法，会被翻译为能直接操作内存而不影响缓存的机器指令。列表 16-26 列出了这些指令。

_**Listing 16-26**.cache\_bypass\_intrinsics.c_

```
#include <emmintrin.h>
void _mm_stream_si32(int *p, int a);
void _mm_stream_si128(int *p, __m128i a);
void _mm_stream_pd(double *p, __m128d a);

#include <xmmintrin.h>
void _mm_stream_pi(__m64 *p, __m64 a);
void _mm_stream_ps(float *p, __m128 a);

#include <ammintrin.h>
void _mm_stream_sd(double *p, __m128d a);
void _mm_stream_ss(float *p, __m128 a);
```

如果我们确定对某块内存区域访问不是长期进行的话，越过缓存是很有用的手段。进一步的信息请参考\[12\]。


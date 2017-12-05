16.2.4 Bypassing Cache

There exists a way to write into memory bypassing cache, which works even in user mode, so in order to use it we should not have access to the page table entries, holdingCWTbit. In Intel 64 an instructionmovntpsallows us to do it. The operating system itself usually sets the bitCWT\(cache-write-through\) in page tables when memory-mapped IO is happening \(when virtual memory acts as an interface for external devices\).  
 In this case, any memory read or write would lead to cache invalidation, which only hinders performance without giving any benefits.

GCC has intrinsic functions that are translated into machine-specific instructions that perform memory operations without involving cache. Listing16-26shows them.

Listing 16-26.cache\_bypass\_intrinsics.c

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

Bypassing cache is useful if we are sure that we will not touch the related memory area for quite a long time. For further information refer to \[12\].




16.4 SSE and AVX Extensions

The SIMD instructions are the basis for the instruction set extensions SSE and AVX. Most of them are used to perform operations on multiple data pairs; for example,mulpscan multiply four pairs of 32-bit floats at once. However, their single operand pair counterparts \(such asmulss\) are now a recommended way to perform all floating point arithmetic.

By default, GCC will generate SSE instructions to operate floating point numbers. They accept operands either inxmmregisters or in memory.

---

**■Consistency** We omit the description of the legacy floating point dedicated stack for brevity. however, we want to point out that all program parts should be translated using the same method of floating point arithmetic: either floating point stack or SSE instructions.

---

We will start with an example shown in Listing16-30.

Listing 16-30.simd\_main.c

```
#include <stdlib.h>
#include <stdio.h>

void sse( float[static 4], float[static 4] );

int main() {
    float x[4] = {1.0f, 2.0f, 3.0f, 4.0f };
    float y[4] = {5.0f, 6.0f, 7.0f, 8.0f };
    
    sse( x, y );
    
    printf( "%f %f %f %f\n", x[0], x[1], x[2], x[3] );
    return 0;
}
```

In this example there is a functionssedefined somewhere else, which accepts two arrays offloats. Each of them should be at least four elements wide. This function performs computations and modifies the first array.

We call the values **packed** if they fill anxmmregister of consecutive memory cells of the same size. In Listing16-30,float x\[4\]is four packed single precision float values.

We will define the functionssein the assembly file shown in Listing16-31.

Listing 16-31.simd\_asm.asm

```
section .text
global sse

; rdi = x, rsi = y

sse:
    movdqa xmm0, [rdi]
    mulps  xmm0, [rsi]
    addps  xmm0, [rsi]
    movdqa [rdi], xmm0
    ret
```

This file defines the functionsse. It performs four SSE instructions:

* movdqa\(MOVeDoubleQwordAligned\) copies 16 bytes from memory pointed byrdi

  into registerxmm0. We have seen this instruction in section 14.1.1.

* mulps\(MULtiplyPackedSingle precision floating point values\) multiplies the contents

  ofxmm0by four consecutive float values stored in memory at the address taken fromrsi.

* addps\(ADD PackedSingled precision floating point\) adds the contents of four

  consecutive float values stored in memory at the address taken fromrsiagain.

* movdqacopiesxmm0into the memory pointed byrdi.

  In other words, four pair of floats are getting multiplied and then the second float of each pair is added to the first one.

The naming pattern is common: the action semantics \(mov, add, mul...\) with suffixes. The first suffix is eitherP\(packed\) orS\(scalar, for single values\). The second one is eitherDfor double precision values \(doublein C\) orSfor single precision values \(floatin C\).

We want to emphasize again that most SSE instructions accept only aligned memory operands.

In order to complete the assignment, you will need to study the documentation for the following instructions using the Intel Software Developer Manual \[15\]:

movsd–Move Scalar Double-Precision Floating- Point Value.

* movdqa–Move Aligned Double Quad word.

* movdqu–Move Unaligned Double Quad word.

* mulps–Multiply Packed Single-Precision Floating Point Values.

* mulpd–Multiply Packed Double-Precision Floating Point Values.

* addps–Add Packed Single-Precision Floating Point Values.

* haddps–Packed Single-FP Horizontal Add.

* shufps–Shuffle Packed Single-Precision Floating Point Values.

* unpcklps–Unpack and Interleave High Packed Double-Precision Floating

  Point Values.

* packswb–Pack with Signed Saturation.

* cvtdq2pd–Convert Packed Dword Integers to Packed Double-Precision FP Values.

These instructions are part of the SSE extensions. Intel introduced a new extension called AVX, which has new registersymm0, ymm1,..., ymm15. They are 256 bits wide, their least significant 128 bits \(lesser half \) can be accessed as oldxmmregisters.

New instructions are mostly prefixed with v, for examplevbroadcastss.

It is important to understand that if your CPU supports AVX instructions it does not mean that they are faster than SSE! Different processors of the same family do not differ by the instruction set but by the amount of circuitry. Cheaper processors are likely to have fewer ALUs.

Let us takemulpswithymmregisters as an example. It is used to multiply 8 pairs of floats.

Better CPUs will have enough ALUs \(arithmetic logic units\) to multiply all eight pairs simultaneously. Cheaper CPUs will only have, say, four ALUs, and so will have to iterate on the microcode level twice, multiplying first four pairs, then last. The programmer will not notice that when using instruction, the semantic is the same, but performance-wise it will be noticeable. A single AVX version ofmulpswithymmregisters and eight pairs of floats can even be slower than two SSE versions ofmuplswithxmmregisters with four pairs each!




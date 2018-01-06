16.4 SSE 和 AVX 扩展

SIMD 指令集是 SSE 和 AVX 扩展的基础。大部分 SIMD 指令都用来操作多个数据对儿；例如，**mulps **可以一次性同时对四对 32 位浮点数进行乘法运算。不过现在在进行浮点数运算时，更推荐单对的操作指令\(例如 **mulss**\)。

默认情况下，GCC 会生成 SSE 指令来操作浮点数。这些指令可以接收 xmm 寄存器或者内存中的内容作为操作数。

---

**■Consistency** 简略起见，我们略过了历史遗留的浮点数专用栈的说明，不过还是需要指出，所有程序部件被翻译的时候都应该使用同一种浮点运算：要么使用浮点数栈，要么使用 SSE 指令集。

---

我们从列表 16-30 中的例子开始吧。

_**Listing 16-30**.simd\_main.c_

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

We call the values **packed** if they fill an xmm register of consecutive memory cells of the same size. In Listing16-30,float x\[4\]is four packed single precision float values.

在列表 16-31 中我们以汇编形式定义一个 sse 函数。

_**Listing 16-31**.simd\_asm.asm_

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

这个文件定义了一个 sse 函数。该函数执行了四种 SSE 指令：

* movdqa\(**MOV**e **D**ouble **Q**word **A**ligned\) 从 rdi 寄存器指向的内存位置拷贝 16 个字节到 xmm0 寄存器。我们在 14.1.1 节中见过这条指令。

* mulps\(**MUL**tiply **P**acked **S**ingle precision floating point values\) 从 rsi 指向的内存地址中取出数据，并对 xmm0 寄存器进行连续四次浮点数运算。

* addps\(**ADD** **P**acked **S**ingled precision floating point\) 从 rsi 指向的内存地址中取数据，并进行四次连续的浮点数计算。

* movdqa 从 xmm0 寄存器中拷贝数据到 rdi 指向的内存地址。

换句话说，四对浮点数会被进行乘法运算，然后每一对的第二个浮点数会再被加到第一个浮点数上。

命名方面的规则倒是比较通用：动作元语\(mov，add，mul...\)加上后缀。第一个后缀可以是 P\(packed\) 或者 S\(scalar 表示单个值\)。第二个后缀可以是 D 表示双精度值\(C 语言里的 double\) 或者 S 表示单精度值\(C 语言里的 float\)。

这里再强调一遍，大多数 SSE 指令只接收进行过内存对齐的内存操作数。

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


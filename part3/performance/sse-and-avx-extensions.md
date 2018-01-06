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

这个地方使用了一个别的地方定义的函数 sse，接收两个 float 类型的数组作为参数。每一个参数都需要至少四个元素。该函数将会执行计算，并修改第一个数组。

我们称多个值为 **packed** 当这些值能够依次填充满同一个 xmm 寄存器。在列表 16-30 中，float x\[4\] 就是四个单精度浮点数 **packed** 的值。

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

为了完成课后作业，你需要查阅 Intel 软件开发手册\[15\] 来学习下面这些指令：

* **movsd**–Move Scalar Double-Precision Floating- Point Value.
* **movdqa**–Move Aligned Double Quad word.
* **movdqu**–Move Unaligned Double Quad word.
* **mulps**–Multiply Packed Single-Precision Floating Point Values.
* **mulpd**–Multiply Packed Double-Precision Floating Point Values.
* **addps**–Add Packed Single-Precision Floating Point Values.
* **haddps**–Packed Single-FP Horizontal Add.
* **shufps**–Shuffle Packed Single-Precision Floating Point Values.
* **unpcklps**–Unpack and Interleave High Packed Double-Precision Floating Point Values.
* **packswb**–Pack with Signed Saturation.
* **cvtdq2pd**–Convert Packed Dword Integers to Packed Double-Precision FP Values.

这些指令是 SSE 扩展的一部分。Intel 引入了名为 AVX 的新扩展，该扩展使用了 ymm0，ymm1...ymm15 等新寄存器。这些寄存器为 256 位宽度，其低 128 位可以当作原来的 xmm 寄存器来使用。

新指令大多数有字母 v 作为前缀，例如 **vbroadcastss**。

了解你的 CPU 是否支持 AVX 很重要，并不是说 AVX 指令一定比 SSE 快！同一产品家族中的处理器在指令集上没有区别，只在集成电路数量上有差别。便宜的处理器一般 ALU 数量会少一些。

结合 mulps 指令和 ymm 寄存器，可以做到同时对 8 对浮点数做乘法。

较好的 CPU 在 ALU 数量上足以同时将八对浮点数同时进行乘法运算。便宜的 CPU 可能只有四个 ALU，所以需要在微指令执行两次操作，每次乘 4 对。程序员一般会注意到使用指令时，可能语义上完全一致，但性能方面却大不相同。AVX 版本的 mulps 和 ymm 寄存器以及 8 对浮点数有时候甚至也可以比 SSE 版本的 mulps 和 xmm 寄存器结合 4 对浮点数运算两次还要慢！


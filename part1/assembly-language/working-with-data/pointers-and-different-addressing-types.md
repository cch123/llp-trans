2.5.4 指针与不同的寻址方式

指针是内存单元的地址。指针的值可以被存在内存或者寄存器中。

指针的大小为 8 个字节。数据一般会占用多个内存单元\(也就是说，几个连续的内存地址\)。指针本身没有存储其指向目标的任何大小信息。如果尝试向一个没有指定大小且没法推断大小的目标写入内容的话\(例如，mov \[myvariable\], 4\)，会发生编译错误。这种情况下我们应该显式地提供大小：

```
section .data
test: dq -1

section .text

mov byte[test], 1 ;1
mov word[test], 1 ;2
mov dword[test], 1 ;4
mov dword[test], 1 ;4
```

---

**■Question 18** 前面的每条指令在执行后，test 的值是什么？

---

我们看看在指令中怎么对指令进行编码。

* 立即数：
  指令本身在内存中。操作数是指令的一部分；这些部分会包含它们自己的地址。很多指令都会包含它们自己的操作数的值。
  下面是将 10 这个数值存进 rax。mov rax, 10。
* 通过寄存器：
  下面的寄存器把 rbx 的值赋给 rax。
  mov rax, rbx
* 通过直接的内存地址：
  这条指令把从地址 10 开始的 8 个字节赋值给 rax：
  mov rax, \[10\]
  也可以直接从寄存器里取内存地址：

```
mov r9，r10
mov rax, [r9]
```

我们还可以使用预计算。

```
buffer: dq 8841, 99, 00
...
mov rax, [buffer+8]

```

这条指令中的地址会被预计算，因为 base 和 offset 都是编译器可以知道的常量。现在这里的地址就只是个数字了。4。

* 基地址扩展法
  很多寻址方式都使用的是这种模式。该模式下的地址计算规则如下：
  `Address = base + index * scale + displacement`
  * base 可以是立即数或者寄存器
  * scale 只能是立即数，值只能是 1，2，4 或者 8
  * index 是立即数或者寄存器；并且
  * displacement 一定是立即数

列表 2-12 展示了不同的寻址类型的示例

_**Listing 2-12**.addressing.asm_

```
mov rax, [rbx + 4* rcx + 9]
mov rax, [4*r9]
mov rdx, [rax + rbx]
lea rax, [rbx + rbx * 4]          ; rax = rbx * 5
add r8, [9 + rbx*8 + 7]
```

整体来看，你可以认为 byte，word 这类的东西是类型标识符。例如，你可以将 16，32 或者 64 位的数字推到栈上去。如果一条指令是 push 1，那么操作数到底有多少个字节是不明确的。同样的情况下，mov word\[test\], 1 则明确地标识了 \[test\] 是一个 word；在  push word 1 这样的指令里，也明确地给出了数值的类型。


16.1.9 执行单元的影响

CPU 由多个部分组成。每条汇编指令都会被分为多阶段执行，每个阶段会由 CPU 的不同部件去完成操作。例如，第一阶段一般被称为指令获取，该阶段需要从内存中加载指令，且不考虑任何语义上的问题。

CPU 中执行操作和计算的部件被称为执行单元。该单元实现了 CPU 想要完成的不同类型的操作：指令获取，算术逻辑运算，地址翻译，指令解码等等。实际上 CPU 某种程度上可以单独使用这个组件了。不同的汇编指令所需要的执行阶段数也是不一样的，每一个阶段都可能会被不同的执行单元所执行。这样就允许一些比较有意思的电路运用了，例如：

* 在获取到另一条指令完成但还没开始执行时，马上去取下一条指令。
* 即使在汇编代码中，按先后顺序描述的算术运算，也会同时执行。

奔腾 IV 家族的 CPU 已经能够在正确的前提下同时执行四个算术指令了。

知道了执行单元的存在之后，我们怎么发挥这样的知识呢？先来看看列表 16-17 中的例子。

_**Listing 16-17**.cycle\_nonpar\_arith.asm_

```
looper:
    mov      rax,[rsi]
    ; The next instruction depends on the previous one.
    ; It means that we can not swap them because
    ; the program behavior will change.
    xor     rax, 0x1

    ; One more dependency here
    add     [rdi],rax
    add     rsi,8
    add     rdi,8
    dec     rcx
    jnz     looper
```

能让这段代码更快么？我们可以分析出影响 CPU 微执行优化器的指令之间的依赖关系。我们展开这个循环，这样老的循环的两次叠代就可以变成新的一次叠代，列表 16-18 展示优化后的结果。

_**Listing 16-18**.cycle\_par\_arith.asm_

```
looper:
mov      rax,  [rsi]
mov      rdx,  [rsi + 8]
xor      rax,  0x1
xor      rdx,  0x1
add      [rdi], rax
add      [rdi+8], rdx
add      rsi, 16

add      rdi, 16
sub      rcx, 2
jnz      looper
```

现在依赖被干掉了，两次叠代的指令没有任何混合了。这样执行起来就可以更快，因为这样的代码能够使 CPU 同时使用不同的执行单元去并发执行两次叠代。指令的依赖应该尽量被优化掉使指令能够同时执行。

---

**■Question 333** 什么是指令流水线和超标量体系架构？

---

我们没法告诉你 CPU 中有哪些执行单元，因为这是和 CPU 的模型相关的。想要获得这些信息只能去阅读特定 CPU 的优化手册，例如 \[16\]。另一些资源可能也会有一些帮助；例如 Haswell 处理器在 \[17\] 中有不错的解读。


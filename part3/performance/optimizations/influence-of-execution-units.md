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

Can we make it faster? We see the dependencies between instructions, which hinder the CPU microcode optimizer. What we are going to do is to unroll the loop so that two iterations of the old loop become one iteration of the new one. Listing16-18shows the result.

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

Now the dependencies are gone, the instructions of two iterations are now mixed. They will be executed faster in this order because it enhances the simultaneous usage of different CPU execution units. Dependent instructions should be placed away from each other to allow other instructions to perform in between.

---

**■Question 333** 什么是指令流水线和超标量体系架构？

---

We cannot tell you which execution units are in your CPU, because this is highly model dependent. We have to read the optimization manuals for a specific CPU, such as \[16\]. Additional sources are often helpful; for example, the Haswell processors are well explained in \[17\].


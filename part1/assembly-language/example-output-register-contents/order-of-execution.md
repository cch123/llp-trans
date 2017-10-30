2.3.3 执行顺序

除了特殊的 jump 指令发生，所有的命令都是被顺序执行的。**jmp addr** 称为无条件跳转指令。其效果可以被认为是 mov rip, addr。

条件跳转则依赖于 rflags 寄存器。例如 jz 地址表示当 zero flag 被设置时跳转。

一般可以使用 test 或者 cmp 指令来设置必须的 flag 和条件跳转指令结合使用。

cmp 指令会从第一个操作数中减去第二个操作数；虽然不在任何地方存储结果，但会根据这个结果设置对应的 flag 信息\(e.g.如果操作数相等，会设置 zero flag\)。test 会做差不多的事情，不过 test 会进行逻辑与操作而不是减法操作。

列表 2-8 的例子包含的代码功能会在rax &lt; 42 时向 rbx 写入 1，否则写入 0。

_**Listing 2-8**.jumps\_example.asm_

```
    cmp rax, 42
    jl yes
    mov rbx, 0
    jmp ex
yes:
    mov rbx, 1
ex:
```

通常使用 test reg, reg 来测试寄存器的值是否为 0，这条指令执行也很快。

每一个算术 flag F 都至少对应了两条命令 F: jF 和 jnF：例如，符号 flag：js 和 jns。其它比较有用的命令包括：

1. ja\(大于时跳转，jump if above\)/jb\(小于时跳转，jump if below\)，两条指令可以和 cmp 结合来比较两个无符号数。
2. jg\(大于时跳转，jump if greater\)/jl\(小于时跳转，jump if less\)，这两条是为有符号数准备的。
3. jae\(大于等于时跳转，jump if above or equal\)，jle\(小于等于时跳转，jump if less or equal\) 类似。一些常见的跳转指令参见列表 2-9。

_**Listing 2-9**.跳转指令:jumps.asm_

```
mov rax, -1
mov rdx, 2

cmp rax, rdx
jg location
ja location        ; different logic!

cmp rax, rdx
je  location       ; if rax equals rdx
jne location       ; if rax is not equal to rdx
```

---

■Question 17 je 和 jz 有啥区别？

---

  



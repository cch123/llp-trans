6.2 中断

中断使我们可以在任意时刻修改程序的控制流。当程序正在执行时，外部事件\(设备需要 CPU 来处理\) 或者内部事件\(除数为零，执行指令的特权级别不够，访问非权威地址\)都会引起中断，而中断会使原本控制流程之外的代码被执行。这段代码叫作**中断处理程序**，中断处理程序一般是操作系统或者驱动程序的一部分。

在 \[15\] 中，Intel 把外部的异步中断和内部的同步异常进行了分离，不过这两种情况的处理方式是类似的。

每一个中断都会被标记一个固定的数值，这个数值就是它的身份标志。对于我们来说处理程序是怎么从中断 controller 中拿到这个中断编号的并不重要。

当第 n 个中断发生时，CPU 会检查内存中的中断描述符表\(IDT\)。和 GDT 类似，其地址和大小是在 idtr 中存储的，图 6-2 描述了这个特殊的寄存器：

_**Figure 6-2**.idtr 寄存器_

IDT 的每一个条目都有 16 字节，第 n 个条目对应第 n 个中断。该条目包含的信息中有例如中断处理器的地址之类。图 6-3 描述了中断描述符的格式。

_**Figure 6-3**.中断描述符_

DPL 描述符特权级别\(Descriptor Privilege Level\)

* 当前特权级别应该要小于等于 DPL 才能使用 int 指令调用中断处理器。否则 check 行为都不会进行。

.1110 类型\(中断门，interrupt gate，IF 标记会在中断处理器中被自动 clear 掉\) 或者 1111 类型\(陷阱门，trap gate，IF 标记不会被 clear\)。

前 30 个中断是被保留的。也就是说你不能自己提供这几个中断的中断处理处理程序，但 CPU 会使用这些保留的中断来处理其内部的一些事件，例如非法的指令编码。除保留中断以外的中断可以被系统程序员使用。

当 IF 标记位被设置时，中断就会被处理；否则中断则会被忽略。

---

■Question 96 什么是 non-maskable 中断？这种中断和编码为 2 的中断和 IF flag 有什么关系？

---

应用程序代码在较低的权限下执行\(ring3\)。直接的设备控制只能在高级的特权级别下执行。当设备需要 CPU 响应并发送中断给 CPU 时，处理程序需要在高特权级别下进行执行，因此需要修改段选择器。

那栈的话呢？这时候栈也应该被切换啊。下面是我们在设置中断描述符的 IST 字段时需要的一些选项：

如果 IST 是 0，那么标准策略会被采用。当中断发生时，ss 寄存器被初始化为 0，新的 rsp 值被加载到 TSS 中。然后 ss 的 RPL 字段被设置为合适的特权级别。之后旧的 ss 和 rsp 值被保存到这个新的栈中。

如果 IST 是 1，那么在 TSS 中的七个 IST 的其中一个会被采用。之所以要创建 IST 是因为发生一些严重的错误\(non-maskable 中断，双重错误等等\)时，在一个已知的栈中执行是可以受益的。所以系统程序员即使是在 ring0 下也会创建一些栈，并使用它们来处理特定的中断。

有一个特殊的 int 指令，这条指令接收中断编号。并依据其描述符内容来调用一段中断处理程序。这个过程会忽略 IF flag：无论 IF 被设置成了什么值，中断处理程序都会被执行。为了控制 int 指令执行的特权代码，就是 DPL 字段的存在意义了。

在中断处理程序开始执行之前，一些寄存器被自动保存进栈。这些寄存器包括 ss，rsp，rflags，cs 和 rip。参见图 6-4 中的栈图。注意段选择器是如何使用零填充到 64 位的。

_**Figure 6-4**.中断处理器开始时的栈情况_

Sometimes an interrupt handler needs additional information about the event. An **interrupt error code** is then pushed into stack. This code contains various information specific for this type of interrupt.

Many interrupts are described using special mnemonics in Intel documentation. For example, the 13-th interrupt is referred to as **\#GP**\(general protection\).1You will find the short description of the some interesting interrupts in the Table6-1.

Table 6-1.Some Important Interrupts

| VECTOR | MNEMONIC | DESCRIPTION |
| :--- | :--- | :--- |
| 0 | \#DE | Divide error |
| 2 |  | Non-maskable external interrupt |
| 3 | \#BP | Breakpoint |
| 6 | \#UD | Invalid instruction opcode |
| 8 | \#DF | A fault while handling interrupt |
| 13 | \#GP | General protection |
| 14 | \#PF | Page fault |

Not all binary code corresponds to correctly encoded machine instructions. Whenripis not addressing a valid instruction, the CPU generates the \#UD interrupt.

The \#GP interrupt is very common. It is generated when you try to dereference a forbidden address \(which does not correspond to any allocated page\), when trying to perform an action, requiring a higher privilege level, and so on.

The \#PF interrupt is generated when addressing a page which has its present flag cleared in the corresponding page table entry. This interrupt is used to implement the swapping mechanism and file mapping in general. The interrupt handler can load missing pages from disk.

The debuggers rely heavily on the \#BP interrupt. When theTFis set inrflags, the interrupt with  
 this code is generated aftereachinstruction is executed, allowing a step-by-step program execution. Evidently, this interrupt is handled by an OS. It is thus an OS’s responsibility to provide an interface for user applications that allows programmers to write their own debuggers.

To sum up, when an n-th interrupt occurs, the following actions are performed from a programmer’s point of view:

1. The IDT address is taken from idtr.
2. The interrupt descriptor is located starting from 128 ×n-th byte of IDT.
3. The segment selector and the handler address are loaded from the IDT entry intocsandrip, possibly changing privilege level. The oldss,rsp,rflags,cs, andripare stored into stack as shown in Figure6-4.
4. For some interrupts, an error code is pushed on top of handler’s stack. It provides additional information about interrupt cause.
5. If the descriptor’stypefield defines it as an Interrupt Gate, the interrupt flagIFis cleared. The Trap Gate, however, does not clear it automatically, allowing nested interrupt handling.

If the interrupt flag is not cleared immediately after the interrupt handler start, we cannot have any kind of guarantees that we will execute even its first instruction without another interrupt appearing asynchronously and requiring our attention.

---

■Question 97 TF flag 会在进入中断处理器时自动被 clear 么？参考 \[15\]。

---

中断处理程序使用 iretq 指令来结束，该指令会恢复之前保存在栈里的所有寄存器，这些寄存器在图 6-4 中有所体现。而简单的 call 指令只会恢复 rip 寄存器。


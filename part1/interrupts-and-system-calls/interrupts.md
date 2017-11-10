6.2 中断

中断使我们可以在任意时刻修改程序的控制流。当程序正在执行时，外部事件\(设备需要 CPU 来处理\) 或者内部事件\(除数为零，执行指令的特权级别不够，访问非权威地址\)都会引起中断，而中断会使原本控制流程之外的代码被执行。这段代码叫作**中断处理器**，中断处理器一般是操作系统或者驱动程序的一部分。

In \[15\], Intel separates external asynchronous interrupts from internal synchronous exceptions, but both are handled alike.

Each interrupt is labeled with a fixed number, which serves as its identifier. For us it is not important exactly how the processor acquires the interrupt number from the interrupt controller.

When then-th interrupt occurs, the CPU checks the Interrupt Descriptor Table\(IDT\), which resides in memory. Analogously to GDT, its address and size are stored inidtr. Figure6-2describes the idtr.

_**Figure 6-2**.idtr 寄存器_

Each entry in IDT takes 16 bytes, and then-th entry corresponds to then-th interrupt. The entry incorporates some utility information as well as an address of the interrupt handler. Figure6-3describes the interrupt descriptor format.

_**Figure 6-3**.中断描述符_

DPL Descriptor Privilege Level

* Current privilege level should be less or equal to DPL in order to call this handler using int instruction. Otherwise the check does not occur.

Type 1110 \(interrupt gate,IFis automatically cleared in the handler\) or 1111 \(trap gate,IFis not cleared\).

The first 30 interrupts are reserved. It means that you can provide interrupt handlers for them, but the CPU will use them for its internal events such as invalid instruction encoding. Other interrupts can be used by the system programmer.

When the IF flag is set, the interrupts are handled; otherwise they are ignored.

---

■Question 96 What are non-maskable interrupts? What is their connection with the interrupt with code 2

and IF flag?

---

The application code is executed with low privileges \(in ring3\). Direct device control is only possible on higher privilege levels. When a device requires attention by sending an interrupt to the CPU, the handler should be executed in a higher privilege ring, thus requiring altering the segment selector.

What about the stack? The stack should also be switched. Here we have several options based on how we set up the IST field of interrupt descriptor.

* If the IST is 0, the standard mechanism is used. When an interrupt occurs,ssis loaded with 0, and the newrspis loaded from TSS. The RPL field ofssthen is set to an appropriate privilege level. Thenoldssandrspare saved in this new stack.

* If an IST is set, one of seven ISTs defined in TSS is used. The reason ISTs are created is that some serious faults \(non-maskable interrupts, double fault, etc.\) might profit from being executed on a known good stack. So, a system programmer might create several stacks even for ring0 and use some of them to handle specific interrupts.

There is a specialintinstruction, which accepts the interrupt number. It invokes an interrupt handler manually with respect to its descriptor contents. It ignores theIFflag: whether it is set or cleared, the handler will be invoked. To control execution of privileged code usingintinstruction, a DPL field exists.

Before an interrupt handler starts its execution, some registers are automatically saved into stack. These aress,rsp,rflags,cs, and rip.See a stack diagram in Figure6-4. Note how segment selectors are padded to 64 bit with zeros.

Figure 6-4.中断处理器开始时的栈情况

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

The interrupt handler is ended by a iretq instruction, which restores all registers saved in the stack, as

shown in Figure6-4, compared to the simplecallinstruction, which restores onlyrip.


6.3.2 syscall and sysret

syscall 指令依赖于几个 MSR\(特殊模块寄存器\) 寄存器。

* STAR \(MSR 编号 0xC0000081\)，该寄存器持有两对儿 cs 和 ss 寄存器的值：一对儿是为系统调用处理器准备的，另一对是为 sysret 指令服务的。图 6-5 展示了它的结构：



Figure 6-5.MSR STAR

* LSTAR \(MSR 编号 0xC0000082\) 持有系统调用处理器的地址\(新的 rip 值\)。

* SFMASK \(MSR 编号 0xC0000084\) 会说明 rflags 寄存器的哪些位应该被系统调用处理器 clear 掉。

syscall 会进行以下操作：

* 从 STAR 中加载 cs 的值。
* 根据 SFMASK 的值修改 rflags。
* 保存 rip 到 rcx；然后
* 用 LSTAR 的值初始化 rip，并从 STAR 中取得新的 cs 和 ss 的值。



Note that now we can explain why system calls and procedures accept arguments in slightly different sets of registers. The procedures accept their fourth argument inrcx, which, as we know, is used to store the oldripvalue.



Contrary to the interrupts, even if the privilege level changes, the stack pointer should be changed by the handler itself.

System call handling ends withsysretinstruction, which loadscsandssfromSTARandripfromrcx.

As we know, the segment selector change leads to a read fromGDTto update its pairedshadow register. However, when executingsyscall, these shadow registers are loaded with fixed values and no reads from GDT are performed.

Here are these two fixed values in deciphered form:

•Code Segment shadow register:

– Base = 0

* – Limit =FFFFFH

* – Type = 112\(can be executed, was accessed\)

* – S = 1 \(System\)

* – DPL = 0

* – P= 1

* – L = 1 \(Long mode\)

* – D= 0

* – G = 1 \(always the case in long mode\)

Additionally, CPL \(current privilege level\) is set to 0

•Stack Segment shadow register:

– Base = 0

* – Limit =FFFFFH

* – Type = 112\(can be executed, was accessed\)

* – S = 1 \(System\)

* – DPL = 0

* – P= 1

* – L = 1 \(Long mode\)

* – D= 1

* – G= 1



However, the system programmer is responsible for fulfilling a requirement: GDT should have thedescriptors corresponding to these fixed values.

So, GDT should store two particular descriptors for code and data specifically forsyscallsupport.


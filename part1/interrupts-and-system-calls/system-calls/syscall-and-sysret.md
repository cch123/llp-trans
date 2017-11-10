6.3.2 syscall and sysret

syscall 指令依赖于几个 MSR\(特殊模块寄存器\) 寄存器。

* STAR \(MSR 编号 0xC0000081\)，该寄存器持有两对儿 cs 和 ss 寄存器的值：一对儿是为系统调用处理器准备的，另一对是为 sysret 指令服务的。图 6-5 展示了它的结构：

![](/assets/6-5.gif)

_**Figure 6-5**.MSR STAR_

* LSTAR \(MSR 编号 0xC0000082\) 持有系统调用处理器的地址\(新的 rip 值\)。

* SFMASK \(MSR 编号 0xC0000084\) 会说明 rflags 寄存器的哪些位应该被系统调用处理器 clear 掉。

syscall 会进行以下操作：

* 从 STAR 中加载 cs 的值。
* 根据 SFMASK 的值修改 rflags。
* 保存 rip 到 rcx；然后
* 用 LSTAR 的值初始化 rip，并从 STAR 中取得新的 cs 和 ss 的值。

我们现在已经可以解释为什么系统调用和方法调用在寄存器的使用上的细微差别了。过程调用的第四个参数是用 rcx 来接收的，而在上面讲到的过程中，rcx 是被用来存储旧的 rip 值了。

和中断相反，即使特权级别发生了变化，栈指针也需要自己中断处理器自行修改。

系统调用处理由 sysret 指令标志结束，这条指令会从 STAR 中加载恢复 cs 和 ss，从 rcx 中恢复 rip。

我们已经知道，段选择器的变化会导致从 GDT 中读取，并将读到的值更新到其对应的 shadow 寄存器中。然而在执行 syscall 时，这些 shadow 寄存器会加载固定的值，不会从 GDT 里读取任何内容。

下面是这两个固定值的解码后的形式：

* 代码段 shadow 寄存器：
  * – Base = 0
  * – Limit =FFFFFH
  * – Type = 112\(can be executed, was accessed\)
  * – S = 1 \(System\)
  * – DPL = 0
  * – P= 1
  * – L = 1 \(Long mode\)
  * – D= 0
  * – G = 1 \(always the case in long mode\)

另外 CPL\(当前特权级别 current privilege level\)被设置为0

* 栈段 shadow 寄存器
  * – Base = 0
  * – Limit =FFFFFH
  * – Type = 112\(can be executed, was accessed\)
  * – S = 1 \(System\)
  * – DPL = 0
  * – P= 1
  * – L = 1 \(Long mode\)
  * – D= 1
  * – G= 1

不管怎么说，系统程序员都需要满足一个需求：GDT 中应该有与这些固定值一致的描述符。

因此，为了支持 syscall，GDT 需要为代码和数据存储两个特殊的描述符。


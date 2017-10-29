1.3.1 General Purpose Registers

程序员大部分时间都在和通用寄存器打交道。通用寄存器本身可互换并且可被不同的命令所用。

从 r0，r1 到 r15 都是 64-bit 的寄存器。前八个寄存器可以有选择性地进行命名，一般其别名的含义会指代其在指令中承担的职责。例如 r1 也被称为 rcx，c 代表着循环\(cycle\)。汇编中的 loop 指令即会使用 rcx 作为循环的计数器，loop 指令本身不接收显式的操作数。当然，这种类型的特殊寄存器在指令中如何使用在文档的指令介绍中都会有所反映\(e.g. 例如这里说的作为 loop 指令的计数器\)。表格 1-2 列出了所有的通用寄存器；同时也可以参考图 1-3。

---

■Note 和主存上实现的硬件栈不同，寄存器是完全不同的另一类内存，因此不像内存单元那样拥有地址

---

可选名实际上是因为一些历史原因而使用更广泛。我们会提供每一个寄存器和其可选名的对照表。其语义也会在描述中给出。现在你还不需要把这些内容记下来。

_**表 1-2** 64 位通用寄存器_

| Name | Alias | Description |
| :--- | :--- | :--- |
| r0 | rax | Kind of an “accumulator,” used in arithmetic instructions. For example, an instruction div is used to divide two integers. It accepts one operand and uses rax implicitly as thesecondone.Afterexecutingdiv rcxabig128-bitwidenumber,storedinpartsin two registers rdx and rax is divided by rcx and the result is stored again in rax. |
| r3 | rbx | Base register. Was used for base addressing in early processor models. |
| r1 | rcx | Used for cycles \(e.g., in loop\). |
| r2 | rdx | Stores data during input/output operations. |
| r4 | rsp | Stores the address of the topmost element in the hardware stack. See section 1.5 “Hardware stack”. |
| r5 | rbp | Stack frame’s base. See section 14.1.2 “Calling convention”. |
| r6 | rsi | Source index in string manipulation commands \(such as movsd\) |
| r7 | rdi | Destination index in string manipulation commands \(such as movsd\) |
| r8 |  |  |
| r9 ... r15 | no | Appeared later. Used mostly to store temporal variables \(but sometimes used implicitly, like r10, which saves the CPU flags when syscall instruction is executed. See Chapter 6 “Interrupts and system calls”\). |

一般情况下不会使用 rsp 和 rbp 这两个寄存器，因为它们有特殊的含义\(之后我们会看它们怎么样破坏掉栈和栈帧\)。不过你依然可以直接用它们来执行算术操作，其实当初这两个寄存器也是为了这个目的而设计的。

表格 1-3 是按照名字顺序进行了排序的一份寄存器列表

_**表 1-3** 64 位通用寄存器---不同的命名约定_

| r0 | r1 | r2 | r3 | r4 | r5 | r6 | r7 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| rax | rcx | rdx | rbx | rsp | rbp | rsi | rdi |

对某个寄存器的其中一部分进行寻址也是可能的。每一个寄存器你可以对其低 32 位，低 16 位或者低 8 位进行寻址。

当使用 r0 ... r15 这样的名字时，只要给寄存器名字加上合适的后缀就可以完成上述操作：

* d 表示 double word --- 即低 32 位
* w 表示 word --- 即低 16 位
* b 表示 byte --- 即低 8 位

例如：

* r7b 表示 r7 的低地址的一个字节
* r3w 表示 r3 的低地址的两个字节
* r0d 表示 r0 的低地址的四个字节

可选的别名同样可以对低字节进行寻址。

图 1-4 展示了如何把通用寄存器分解为更小的部分。

rax，rbx，rcx 和 rdx 这几个寄存器的命名约定是一致的；区别只有中间这个字母\(rax 中的 a\)。另外的四个寄存器则不能访问其低位的两个字节\(就像你用 ah 来访问 rax 的一部分那样\)。rsi，rdi，rsp 和 rbp 这几个寄存器的低字节访问命名规则有一些细微的差别。

* rsi 和 rdi 的低字节部分命名分别是 sil 和 dil\(见图 1-5\)

* rsp 和 rbp 的低字节部分命名分别是 spl 和 bpl\(见图 1-6\)

实践中 r0 - r7 这几个名字很少见。程序员一般都会使用其别名来使用前八个寄存器。无论是从历史还是语义上来讲，rsp 这样的命名都比 r4 包含了更丰富的信息。至于后八个寄存器\(r8-r15\)则只能使用索引式的命名法。

---

■Inconsistency in writes 写时不一致。从寄存器的低位读取数据操作是很直观的。不过如果你想要写一个寄存器的低 32 位，那么其高 32 位也会被符号位的值填充。例如把 eax 置 0 会让 rax 也整个变成 0。往 eax 里存 -1 的话，那高 32 位会全部被填 1。除了这两种情况的其它写\(e.g.往低 16 位写\)则和预期相符：即低位写不会影响到高位。这个现象的详细解释参见 3.4.2 章节

---




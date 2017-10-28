1.1.2 冯诺伊曼架构

现在让我们想象自己还活在 1930 年代，今天电脑还没有出现。人们想要把计算过程自动化，研究员们想出了各种法子来达到目的。常见的例如 Church 的 Lambda 演算或者图灵机器。这些都是典型的抽象机器，描述了假想的计算机形式。

很快一种机器就变成了主流：冯诺伊曼架构的计算机。

计算机架构一般会描述一台机器的功能，组织和系统的实现。相比计算模型来说是较高层级的描述，而后者则重点都在各种细粒度的描述上。

冯诺伊曼架构最关键的进步有两点：一是其鲁棒性\(要知道这是一个电子元件非常不稳定而且快速损坏的世界\)，二是其编程的简易性。

简单来说，这是一台包含了一个处理器和一个内存“银行”，且被连接到一个通用的总线上的计算机。中央处理器\(CPU\)可以执行指令，这些指令是用控制单元\(control unit\)从内存中获取到的。计算逻辑单元\(ALU\)的职责则是进行计算。内存里除指令之外还会存储数据，参见图 1-1 和图 1-2。

下面是该架构的几个核心特性：

* 内存只存储 bit \(一个信息单元，取值范围是 0 或者 1\)
* 内存既存储编码后的指令，也存储需要操作的数据。从表现形式上没有办法区分数据和代码：因为二者皆是以 bit 串的形式存储
* 内存以单元进行组织，这些单元以其相对位置进行自然标记\(e.g. 单元 \#43 跟在单元 \#42 之后\)。索引是从 0 开始，单元的大小不一\(冯诺伊曼认为每一个 bit 都有对应的地址\)；不过现代计算机实际上是以 byte \(8 bit\) 作为内存单元大小的。因此，第 0 个字节指向的位置也是内存中的最前面 8 个 bit 的内存
* 程序由指令组成，指令是一条一条按顺序获取。指令的的执行也是按顺序进行，除非有 jump 这样的指令被执行

![](/assets/1-1.jpg)

图 1-1.冯诺伊曼架构--总览

Assembly language for a chosen processor is a programming language consisting of mnemonics for each possible binary encoded instruction \(machine code\). It makes programming in machine codes much easier, because the programmer then does not have to memorize the binary encoding of instructions, only their names and parameters.

Note, that instructions can have parameters of different sizes and formats.

An architecture does not always define a precise instruction set, unlike a model of computation.

A common modern personal computer have evolved from old von Neumann architecture computers,so we are going to investigate this evolution and see what distinguishes a modern computer from the simple schematic in Figure1-2.

![](/assets/1-2.jpg)

图 1-2.冯诺伊曼架构---内存

---

■Note memory state and values of registers fully describe the Cpu state \(from a programmer’s point of

view\). understanding an instruction means understanding its effects on memory and registers.

---




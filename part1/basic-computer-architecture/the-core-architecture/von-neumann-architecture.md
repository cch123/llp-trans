1.1.2 冯诺伊曼架构

现在让我们想象自己还活在 1930 年代，今天电脑还没有出现。人们想要把计算过程自动化，研究员们想出了各种法子来达到目的。常见的例如 Church 的 Lambda 演算或者图灵机器。这些都是典型的抽象机器，描述了假想的计算机形式。

很快一种机器就变成了主流：冯诺伊曼架构的计算机。

计算机架构一般会描述一台机器的功能，组织和系统的实现。相比计算模型来说是较高层级的描述，而后者则重点都在各种细粒度的描述上。

冯诺伊曼架构最关键的进步有两点：一是其鲁棒性\(要知道这是一个电子元件非常不稳定而且快速损坏的世界\)，二是其编程的简易性。

简单来说，这是一台包含了一个处理器和一个内存“银行”，且被连接到一个通用的总线上的计算机。中央处理器\(CPU\)可以执行指令，这些指令是用控制单元\(control unit\)从内存中获取到的。计算逻辑单元\(ALU\)的职责则是进行计算。内存里除指令之外还会存储数据，参见图 1-1 和图 1-2。

下面是该架构的几个核心特性：

内存只存储比特\(一个信息单元，取值范围是 0 或者 1\)

Memory stores both encoded instructions and data to operate on. There are no means to distinguish data from code: both are in fact bit strings.

* Memory is organized into cells, which are labeled with their respective indices in  
   a natural way \(e.g., cell \#43 follows cell \#42\). The indices start at 0. Cell size may  
   vary \(John von Neumann thought that each bit should have its address\); modern computers take one byte \(eight bits\) as a memory cell size. So, the 0-th byte holds the first eight bits of the memory, etc.

* The program consists of instructions that are fetched one after another. Their execution is sequential unless a special jump instruction is executed.

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




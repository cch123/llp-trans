3.4.2 CISC and RISC

处理器有一种按照其指令集进行分类的方式。当设计一个处理器时有两个极端。

* 设计出各种特化指令，高级指令。这种架构叫作 **CISC \(Complete Instruction Set Computer\) **架构。
* 只使用一些基本指令，完成的架构叫 **RISC \(Reduced Instruction Set Computer\) **架构。

CISC 指令一般运行比较慢而且会做更多的事情；有时候实现比较复杂的指令也可以比需要组合一大堆 RISC 指令来说更方便\(这本书在之后 16 章介绍 SSE \(Streaming SIMD Extensions\)的时候会有一个例子\)。然而，大多数高级语言的程序是依赖编译器的，如果想要实现一个 RISC 指令集的编译器非常得困难。

RISC 简化了编译器的工作，而且对于低级，微代码级别的优化非常友好，比如 pipeline。




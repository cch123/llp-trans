14.1.1 XMM 寄存器

除了我们已经讨论过的寄存器，现代处理器还有一些扩展。这些扩展体现在电路上，指令集上，有时候也会扩展一些很有用的寄存器。比较著名的扩展叫作 SSE \(Streaming SIMD Extensions\)，该扩展加入了新的 xmm 寄存器集合：xmm0，xmm1，...，xmm15。这些寄存器为 128 位宽，常用于两种任务：

* 浮点数运算；以及
* SIMD 指令集\(这种指令一条指令可以操作多条数据\)

常用的 mov 指令没有办法操作 xmm 寄存器。movq 指令可以代替用来拷贝 xmm 寄存器的低位\(128 位中的低 64 位\)，操作数的其中一个可以也是 xmm 寄存器，或者通用寄存器，或者内存\(也得是 64 位\)。

为了填满 xmm 寄存器，你有两个选择：movdqa 和 movdqu。前者可以解释为“ move aligned double quad word”，移动两个对齐的 qword。后者是未对齐的版本。

大多数 SSE 指令都需要内存操作数适当地进行对齐。上面说到的未对齐版本的指令和对齐版本的指令在助记符上就有差别，而且因为内存未对齐的关系，性能也会受影响。由于 SSE 指令经常被用在性能敏感的场合，所以始终使用操作数内存对齐版本的指令是明智之举。

我们将在 16.4.1 节中使用 SSE 指令来执行高性能计算任务。

---

■Question 263 阅读 \[15\] 中关于 movq，movdqa 和 movdqu 指令的资料。

---




16.5 小结

本章我们讨论了编译器优化以及每种优化的必要性。我们学习到了翻译并优化后的代码和最初的版本能有多么大的区别。然后研究了如何从缓存中最大程度获益，以及使用 SSE 扩展，在指令级别进行浮点数运算的并行化。下一章中我们将学习如何并行化指令序列的执行，创建多个线程，并以此来刷新我们多线程方面的知识。

---

■Question 336 哪些 GCC 选项是全局优化控制选项？

■Question 337 哪种类型的潜在优化可以带来最大的益处？

■Question 338 省略栈帧指针的优点和缺点分别是什么？

■Question 339 尾递归函数和普通的递归函数有什么区别？

■Question 340 一个递归函数可以在不使用更多的数据结构的前提下被改写为尾递归形式么？

■Question 341 What is a common subexpression elimination? how does it affect our code writing?

■Question 342 什么是常量传播？

■Question 343 为什么我们要尽可能地将函数标记为 static 来帮助编译器优化？

■Question 344 命名函数返回优化有什么好处？

■Question 345 什么是分支预测？

■Question 346 什么是动态分支预测，什么是全局和局部的历史表？

■Question 347 检查你的 CPU 在分支预测上的相关笔记\[16\]。

■Question 348 什么是执行单元，我们为什么要关注执行单元？

■Question 349 AVX 指令的速度和执行单元的数量有啥关系？

■Question 350 哪种内存访问模型对性能来说比较好？

■Question 351 为什么我们需要这么多的缓存级别？

■Question 352 哪些情况下可以通过预取获得性能提升，为什么？

■Question 353 SSE  指令是干什么用的？

■Question 354 为什么大多数 SSE 指令都要求操作数对齐？

■Question 355 如何从通用寄存器中拷贝数据到 xmm 寄存器？

■Question 356 什么情况下值得使用 SIMD 指令来做优化？

---




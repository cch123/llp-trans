在本章中我们学习了虚拟内存实现的概念。我们以 cache 的 case 对其进行了详细说明。然后回顾了不同类型的地址空间\(物理地址空间、虚拟地址空间\)和它们之间的联系\(几组地址翻译表\)。然后一直深入到虚拟内存的实现细节当中。

最后我们提供了一个最小可工作的样例，使用 linux 系统调用来演示内存映射。我们会在 Chapter 13 重新用到它，在那个章节中会基于这个工具来实现我们的动态内存分配器。下一章我们将会学习程序的翻译和链接过程，来理解操作系统如何使用虚拟内存技术来载入并执行程序。

Question 50

什么是虚拟内存 region？

Question 51

如果在程序运行时尝试修改程序代码会发生什么事情？

Question 52

什么是禁用地址？

Question 53

什么是权威地址？

Question 54

什么是翻译表？

Question 55

What is a page frame?

Question 56

什么是内存 region？

Question 57

虚拟内存空间是什么？和物理的有啥不同？

Question 58

What is a Translation Lookaside Buffer?

Question 59

What makes the virtual memory mechanism performant?

Question 60

How is the address space switched?

Question 61

Which protection mechanisms does the virtual memory incorporate?

Question 62

What is the purpose of EXB bit?

Question 63

What is the structure of the virtual address?

Question 64

Does a virtual and a physical address have anything in common?

Question 65

Can we write a string in .text section? What happens if we read it? And if we overwrite it?

Question 66

Write a program that will call stat, open, and mmap system calls \(check the system calls table in Appendix C\). It should output the file length and its contents.

Question 67

Write the following programs, which all map a text file input.txt containing an integer x in memory using a mmap system call, and output the following:

1. x! \(factorial, x! = 1 · 2 · · · · · \(x − 1\) · x\). It is guaranteed that x ≥ 0.

2. 0 if the input number is prime, 1 otherwise.

3. Sum of all number’s digits.”

4. x-th Fibonacci number.

5. Checks if x is a Fibonacci number.

Footnotes

1 An interrupt \#PF \(Page Fault\) occurs.

2 To find the process identifier, use such standard programs as ps or top.

3 Theoretically we could support all 64 bits of physical addresses, but we do not need that many addresses yet.”


2.8 小结

本章我们开始写真实的代码并将我们的基础知识应用于汇编语言。希望你能克服对汇编的恐惧。尽管可能有些冗长得极端，但并不是一门很难使用的语言。我们已经学会了如何使用分支和循环来完成基本的算术逻辑和系统调用；也学习了不同的寻址模式，大小端。接下来的作业会使用到我们前面建立的和用户交互的库。

---

■Question 23 rax，eax，ax，ah 和 al 之间有什么联系？

■Question 24 如何访问 r9 的一部分位？

■Question 25 如何在硬件栈上工作？列出你需要用到的指令。

■Question 26 下面哪些指令是不正确的，为什么？

```
mov [rax], 0
cmp [rdx], bl
mov bh, bl
mov al, al
add bpl, 9
add [9], spl
mov r8d, r9d
mov r3b, al
mov r9w, r2d
mov rcx, [rax + rbx + rdx]
mov r9, [r9 + 8*rax]
mov [r8+r7+10], 6
mov [r8+r7+10], r6
```

■Question 27 枚举 callee-saved 寄存器

■Question 28 枚举 caller-saved 寄存器

■Question 29 rip 寄存器用来做什么？

■Question 30 SF flag 表示什么含义？

■Question 31 ZF flag 表示什么含义？

■Question 32 描述下列指令产生的效果：

* sar
* shr
* xor
* jmp
* ja, jb and similar ones...
* cmp
* mov
* inc,dec
* add
* imul, mul
* sub
* idiv, div
* call, ret
* push, pop

■Question 33 什么是 label，label 有大小么？

■Question 34 如何检查一个整数是否在范围 \(x, y\) 之内？

■Question 35 ja/jb 和 jg/jl 的区别是什么？

■Question 36 je 和 jz 之间有什么区别？

■Question 37 如何在不使用 cmp 的前提下检查 rax 是不是零？

■Question 38 什么是程序的返回码？

■Question 39 如何使用一条指令给 rax 乘 9？

■Question 40 使用两条指令\(第一条是 neg\)，获取 rax 中存储的值的绝对值。

■Question 41 大端和小端之间有什么区别？

■Question 42 最复杂的寻址方式是什么？

■Question 43 程序执行的入口在哪里？

■Question 44 rax=0x112233445567788。我们执行了 push rax 指令。那么现在地址 \[rsp + 3\] 中的内容是什么？

---




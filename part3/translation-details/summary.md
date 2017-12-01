14.9 小结

本章复习了 Intel 64 平台 \*nix 系统上的调用规约。并且学习了一些使用 C 的高级特性的几个例子，比如 volatile，restrict 和非本地跳转。最后我们简单总结了由于栈帧的组织而导致程序可能被攻击到的弱点，以及编译器自动对付这些攻击手段的特性。下一章我们会解释更多的动态链接库底层细节，来加强我们对这些技术的理解。

---

■Question 267 xmm 寄存器是什么？总共有多少个这样的寄存器？

■Question 268 SIMD 指令是什么？

■Question 269 为什么一些 SSE 指令需要内存操作数是对齐的？

■Question 270 为什么寄存器被用来向函数传递参数？

■Question 271 向函数传递参数时，为什么 rax 偶尔会被用到？

■Question 272 rbp 寄存器一般是怎么使用的？

■Question 273 栈帧是什么？

■Question 274 为什么我们不使用 rsp 来相对地定位局部变量？

■Question 275 什么是 prologue 和 epilogue？

■Question 276 enter 和 leave 指令的意图是什么？

■Question 277 讲讲函数执行期间的栈帧是如何变化的？

■Question 278 什么是 red zone？

■Question 279 如何定义和使用变长参数的函数？

■Question 280 va list 持有什么样的上下文？

■Question 281 为什么需要使用 vfprintf 这样的函数？

■Question 282 volatile 变量的意图是什么？

■Question 283 为什么只有 volatile 类型的栈变量在 longjmp 之后会存活？

■Question 284 所有的局部变量都是在栈上分配空间的吗？

■Question 285 setjmp 是用来干啥的？

■Question 286 setjmp 的返回值是什么？

■Question 287 restrict 是干啥用的？

■Question 288 restrict 可以被编译器忽略吗？

■Question 289 如何在不使用 restrict 的前提下，获得和 restrict 相同的效果？

■Question 290 解释使栈缓冲溢出的手段。

■Question 291 使用时候使用 printf 是不安全的？

■Question 292 什么是 security cookie？这种手段可以避免缓冲区溢出时的程序崩溃么？

---




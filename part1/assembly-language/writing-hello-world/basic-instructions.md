2.2.3 基础指令

mov 指令用来将一个值写入到寄存器或内存。这里的“值”可以是从其它寄存器或者从内存中取得，或者也可以是一个立即数。不过还有一些限制：

1. mov 不能把数据从内存拷贝到内存
2. 数据源和数据目标的操作数大小必须相等

syscall 指令在 \*nix 系的系统中进行系统调用。输入输出操作依赖于硬件设备，但这些硬件可被多个程序同时使用，因此不允许程序员越过操作系统直接使用。

每一个系统调用都会有一个唯一的编号。如果想要调用它：

1. rax 寄存器中需要存该系统调用编号
2. rdi，rsi，rdx，r10，r8 和 r9 这些寄存器依次存储传入的参数
   系统调用不能超过六个参数
3. 执行 syscall 指令

初始化寄存器的顺序无所谓。

需要注意的是，系统调用过程中把 rcx 和 r11 两个寄存器换掉了！之后我们会解释原因。当我们写 "Hello, world!" 程序的时候用到了 write 的系统调用。其接收的参数为：

1. 文件描述符
2. buffer 地址。我们可以从这里开始获取连续字节
3. 需要写的字节数

为了编译我们第一个程序，将代码保存到 **hello.asm** 然后在 shell 里执行这些命令：

```
> nasm -felf64 hello.asm -o hello.o
> ld -o hello hello.o
> chmod u+x hello
```

编译过程经过的多个步骤会在第五章中详细讨论。让我们运行 “Hello, world!”。

```
> ./hello
hello, world!
Segmentation fault
```

我们确实获取到了想要的输出，不过程序似乎还是出了点问题。我们哪里做错了呢？在执行完 syscall 之后，程序会继续执行。但我们在 syscall 之后又没有写任何指令，不过实际上之后的内存单元中会有一些完全随机的内容。

---

**■Note** 如果你不在某个内存地址里写内容，那么这段内存里就会一些垃圾数据，不是零，也不是什么非法的指令。

---

处理器对于这些值到底是不是编码后的机器码并不清楚。因此按照 CPU 的特性，它还是会尝试翻译这些值，因为寄存器指向了这些值。自然这些指令大概率不是合法的指令，所以一个编号为 6 的中断就会发生了\(非法指令\)。

那么我们怎么办呢？我们需要在程序中使用 exit 这个系统调用，该系统调用能够正确地结束程序，像列表 2-4 展示的一样：

_**Listing 2-4**.hello\_proper\_exit.asm_

```
section .data
message: db 'hello, world!', 10

section .text
global _start

_start:
    mov     rax, 1              ; 'write' syscall number
    mov     rdi, 1              ; stdout descriptor
    mov     rsi, message        ; string address
    mov     rdx, 14             ; string length in bytes
    syscall
    
    
    mov     rax, 60             ; 'exit' syscall number
    xor     rdi, rdi
    syscall

```

---

■Question 11 xor rdi, rdi 这条指令做了什么事情？

■Question 12 什么是程序的返回码？

■Question 13 exit 系统调用的强一个参数是什么？

---




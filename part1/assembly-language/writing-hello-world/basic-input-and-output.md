2.2.1 基本输入输出

Unix 的奉行“一切皆文件”的哲学。**文件**，是一个比较大的概念，是看起来像字节流的东西。从文件中可以抽象出一些概念，例如：

* 访问硬盘/SSD 上的数据
* 在程序间进行数据交换；以及
* 和外部设备交互

我们还是遵循传统，从写一个简单的 ”Hello, world!“ 程序开始。这个程序在屏幕上打印一条欢迎信息然后退出。不管怎么说，这个程序需要在屏幕上显示字符，如果在没有操作系统的裸机上做不太容易实现。操作系统的功能之一即是抽象并管理资源，然后显示某些资源。系统提供了一些常规程序集合来处理与外部设备、其它程序、文件系统等内容的通信。一个程序没办法越过操作系统和这些受操作系统控制的资源交互。如果想要交互，那也只能通过系统调用来完成，系统调用即前面提到的系统提供的“常规程序集合”。

Unix 用描述符来描述文件，当文件被程序打开时即会返回这个描述符。描述符本身没什么特殊的，就是一个普通的整型数值\(例如 42 或者 999\)。显式地调用 open 这个系统调用就可以打开文件；不过也有例外，有三个特殊的文件在程序启动时即会被打开而且你也不应该手动处理这些文件。这三个特殊的文件是 stdin，stdout，stderr。它们的描述符分别是 0，1 和 2。stdin 用来处理输入，stdout 处理输出，stderr 处理程序运行过程中的信息，但这些信息不是程序本身的运行结果\(e.g.例如错误和诊断信息\)。

默认情况下，键盘的输入和 stdin，终端输出和 stdout 会进行绑定。也就是说我们的程序的 "Hello, world!" 应该写入到 stdout 中。

因此我们还需要调用 write 这个系统调用。该系统调用会从内存读取指定大小的字节并写入到给出了描述符\(我们这里的描述符是 1\)的文件中去。这些字节会按照 ascii 表对字符进行编码。每一个单元即是一个字符；而每一个字符在 ascii 表中都有 0-255 的唯一编码与之对应。

列表 2-1 是我们第一个完整的汇编程序样例

_**Listing 2-1**.hello.asm_

```
global _start

section .data
message: db 'hello, world!', 10

section .text
_start:
    mov     rax, 1           ;system call number should be stored in rax
    mov     rdi, 1           ; argument #1 in rdi: where to write (descriptor)?
    mov     rsi, message     ; argument #2 in rsi: where does the string start?
    mov     rdx, 14          ; argument #3 in rdx: how many bytes to write?
    syscall                  ; this instruction invokes a system call

```

这个程序调用了 write 系统调用，并通过代码 6-9 行传入了正确的参数。也就只做了这一件事情。下一小节会详细地解释这个程序的细节。


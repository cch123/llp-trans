4.5 示例：访问禁用地址

现在我们将用自己的眼睛，来看看一个单进程的内存布局。示例将会展示哪些页当前可用并且对应什么内容。我们将观察不同类型的内存 region：

1. 对应可执行文件，自己载入到内存
2. 对应核心库
3. 对应栈和堆\(匿名页\)
4. 只是禁用地址的空的区域

Linux 提供了一种简单的手段帮助我们探索各种各样的信息，关于进程的叫作 **procfs**。其提供了一套特殊目的的文件系统，通过浏览路径和文件，我们就可以获知任意进程的内存，环境变量等信息。这套文件系统被挂载在 /proc 路径。

特别是文件 /proc/PID/map 会展示一个进程 ID 为 PID 的内存布局。

我们来写一个简单的程序，该程序进入循环后不再退出，见列表 4-1。这样就可以让我们在这个程序运行的时候查看其内存布局。

_**Listing 4-1**.mappings\_loop.asm_

```
section .data
correct: dq -1
section .text

global _start
_start:
jmp _start
```

现在我们查看 /proc/?/maps，? 代表进程 ID。完整命令行内容参见列表 4-2。

_**Listing 4-2**.mappings\_loop_

```
> nasm -felf64 -o main.o mappings_loop.asm
> ld -o main main.o
> ./main &
[1] 2186
> cat /proc/2186/maps
00400000-00401000 r-xp 00000000 08:01 144225 /home/stud/main
00600000-00601000 rwxp 00000000 08:01 144225 /home/stud/main
7fff11ac0000-7fff11ae1000 rwxp 00000000 00:00 0 [stack]
7fff11bfc000-7fff11bfe000 r-xp 00000000 00:00 0 [vdso]
7fff11bfe000-7fff11c00000 r--p 00000000 00:00 0 [vvar]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0 [vsyscall]
```

左边的列定义了内存 region 的范围。你可能注意到了，所有的 region 都被包含在以三个十六进制 0 结尾的地址中。这是由于这些内存是以页形式组织的，每个页都是 4KB\(= 0x1000 字节\)。

我们还观察到，汇编文件中定义的不同的段会被载入到不同的 region。第一个 region 对应代码段，存储了编码后的指令；第二个段对应数据段。

就像你看到的，地址空间是巨大的，且一直从第 0 字节地址扩展到 2^64 -1 字节的地址。但是，只有一部分地址会被真正分配；其它地址都是禁止访问的。

第二列展示了页的读、写和执行权限。也表明了页是否在多进程间共享或者其对特定的进程是 private。

---

■Question 48 阅读 procfs 的 man 手册的第四 \(08:01\) 和第五 \(144225\) 列

---

目前我们没有什么做错的地方。现在我们尝试向一段禁止的区域写入内容。

_**Listing 4-3**.产生 segfault:segfault\_badaddr.asm_

```
section .data
correct: dq -1
section .text
global _start
_start:
mov rax, [0x400000-1]
; exit
mov rax, 60
xor rdi, rdi
syscall

```

我们访问 0x3fffff 这个地址，这段地址是代码段之前一个字节的地址。这个地址是被禁止访问的地址，因此写入操作一定会引发段错误，有下面这样的信息提示。

```
> ./main Segmentation fault
```




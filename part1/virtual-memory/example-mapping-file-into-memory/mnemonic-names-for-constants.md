4.9.1 常量的助记名

Linux 是用 C 写的，为了和操作系统交互方便，一般会用 C 风格的方式来预定义一些常量。

```
#define NAME 42
```

这一行定义了编译时替换的常量。无论程序员在什么地方写了 NAME ，编译器都会将其替换为 42。这种特性可以让我们方便地为各种常量定义助记名。NASM 也提供了类似的功能，像下面：

```
%define directive
%define NAME  42
```

参考 5.1 节 “预处理器”，来获知这种替换是如何进行的。

我们来看看系统调用 mmap 的 man 手册中对第三个参数 prot 的说明。

prot 参数描述了映射时想要的内存保护\(该保护级别不能与调用 open 打开文件时的 mode 参数所冲突\)。参数的值可以是 `PROT_NONE` 或者是下面几个 flags 的按位或：

```
PROT_EXEC Pages may be executed.
PROT_READ Pages may be read.
PROT_WRITE Pages may be written.
PROT_NONE  Pages may not be accessed.

```

PROT\_NONE 和其它的几个参数就是上面提到的助记名的例子，每一个助记名都代表了一个 integer 值，这些值用来控制 mmap 的行为。C 和 NASM 都允许你在编译期执行一些常量的值计算，包括按位与/按位或操作。下面是这种计算的一个例子。



```
%define PROT_EXEC 0x4
%define PROT_READ 0x1
mov rdx, PROT_READ | PROT_EXEC
```





Unless you are writing in C or C++, you will have to check these predefined values somewhere and copy them to your program.



Following is how to know the specific values of these constants for Linux:



Search them in header files of the Linux API in/usr/include.

Use one of the Linux Cross Reference \(lxr\) online, like:http://lxr.free-electrons.com.



We do recommend the second way for now, as we do not know C yet. You may even use a search engine like Google and type lxr  PROT\_READ as a search query to get relevant results immediately after following the first link.





For example, here is what LXR shows when being queriedPROT\_READ:



PROT\_READ

```
Defined as a preprocessor macro in:
```

```
arch/mips/include/uapi/asm/mman.h, line 18
arch/xtensa/include/uapi/asm/mman.h, line 25
arch/alpha/include/uapi/asm/mman.h, line 4
arch/parisc/include/uapi/asm/mman.h, line 4
include/uapi/asm-generic/mman-common.h, line 9

```

By following one of these links you will see

18 \#define PROT\_READ 0x01 /\* page can be read \*/  


So, we can type%define PROT\_READ 0x01in the beginning of the assembly file to use this constant

without memorizing its value.  



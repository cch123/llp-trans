2.5.1 大小端

  
让我们用刚刚写的方法来输出内存中存储的值。我们会用两种不同的方法来达到这个目的：第一个种会分开输出它的各个字节，第二种则是按照正常整体输出。参见列表 2-11：

Listing 2-11.endianness.asm

```
section .data
demo1: dq 0x1122334455667788
demo2: db 0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88

section .text

_start:
    mov rdi, [demo1]
    call print_hex
    call print_newline
    
    mov rdi, [demo2]
    call print_hex
    call print_newline
    
    mov rax, 60
    xor rdi, rdi
    syscall
    
```

运行这个程序，输出的结果可能让我们有点儿意外，我们在 demo1 和 demo2 中得到了完全不同的结果。

```
> ./main
1122334455667788
8877665544332211
```

可以看到，多字节的数值是以相反的顺序进行存储的！

每一个字节内的数值顺序倒是比较直观，但是多个字节则是从小一直存储到大了。

这种场景只适用于内存操作：在寄存器中的字节存储是比较自然的顺序。不同的处理器对于数值的存储方式有不同的约定。

* **大端模式**下多字节数是从最大的字节开始存储的。
* **小端模式**下多字节数是从最小的字节开始存储的。

像示例中展示的一样，Intel 64 遵循的是小端的规范。一般来说，选择某种规范只是硬件工程师的一个选择。

上面的约定和字符串或者数组的存储方式没有什么关系。不过如果每一个字符都是由两个而不是一个字节编码而成的话，这些字节会被以反序存储。



The advantage of little endian is that we can discard the most significant bytes effectively converting the number from a wider format to a narrower one, like 8 bytes.

For example,demo3: dq 0x1234. Then, to convert this number intodwwe have to read a dword number starting at the same addressdemo3. See Table2-1for a complete memory layout.

Table 2-1.Little Endian and Big Endian for quad word number 0x1234

| ADDRESS | VALUE-LE | VALUE-BE |
| :--- | :--- | :--- |
| demo3 | 0x34 | 0x00 |
| demo3 + 1 | 0x12 | 0x00 |
| demo3 + 2 | 0x00 | 0x00 |
| demo3 + 3 | 0x00 | 0x00 |
| demo3 + 4 | 0x00 | 0x00 |
| demo3 + 5 | 0x00 | 0x00 |
| demo3 + 6 | 0x00 | 0x12 |
| demo3 + 7 | 0x00 | 0x32 |

Big endian is a native format often used inside network packets \(e.g., TCP/IP\). It is also an internal number format for Java Virtual Machine.

Middle endianis a not very well-known notion. Assume we want to create a set of routines to perform arithmetic with 128-bit numbers. Then the bytes can be stored as follows: first will be the 8 least significant bytes in reversed order and then the 8 most significant bytes also in reverse order:

7 6 5 4 3 2 1 0, 16 15 14 13 12 11 10 9 8












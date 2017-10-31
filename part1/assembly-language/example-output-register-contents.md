2.3 示例：输出寄存器内容

来试试更难的程序吧。让我们以十六进制的形式把 rax 的值进行输出，像列表 2-5 这样：

_**Listing 2-5.**打印 rax 的值：print\_rax.asm_

```
section .data
codes:
    db      '0123456789ABCDEF'

section .text
global _start
_start:
    ; number 1122... in hexadecimal format
    mov rax, 0x1122334455667788

    mov rdi, 1
    mov rdx, 1
    mov rcx, 64
   ; Each 4 bits should be output as one hexadecimal digit
   ; Use shift and bitwise AND to isolate them
   ; the result is the offset in 'codes' array
.loop:
    push rax
    sub rcx, 4
   ; cl is a register, smallest part of rcx
   ; rax -- eax -- ax -- ah + al
   ; rcx -- ecx -- cx -- ch + cl
    sar rax, cl
    and rax, 0xf

    lea rsi, [codes + rax]
    mov rax, 1
   ; syscall leaves rcx and r11 changed
    push rcx
    syscall
    pop rcx

    pop rax
   ; test can be used for the fastest 'is it a zero?' check
   ; see docs for 'test' command
    test rcx, rcx
    jnz .loop
    mov     rax, 60 ;          invoke 'exit' system call
    xor      rdi, rdi
    syscall
```

通过对 rax 的值进行移位以及将其与 0xF 进行位与运算，可以将整个数字转换成它的十六进制数形式。每一个十六进制字符都是 0 到 15 的数字。使用这个数字作为索引即可在字符串中查询到其对应的字符。

例如，rax = 0x4A。我们使用 0x4 = 4 和 0xA = 10，第一个索引查到的字符是 '4'，值是 0x34。第二个字符是 'a'，值为 0x61。

---

■Question 14 检查例子中用到的 ASCII 代码是否正确。

---

我们可以用硬件栈来保存和恢复寄存器的值，就像 syscall 周围代码所做的一样。

---

■Question 15 sar 和 shr 之间的区别是什么？请查阅 Intel 的文档。

■Question 16 如何在不同的数值系统下以 nasm 可以理解的形式书写数字呢？请查阅 nasm 的文档。

---

---

■Note 程序启动时，大多数寄存器的值还没有被初始化\(内容可能是完全随机的\)。假设已被清零，是新手最容易犯的错误。

---




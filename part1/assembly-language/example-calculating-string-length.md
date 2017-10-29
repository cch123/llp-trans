2.6 示例：计算字符串长度

让我们先来写一个计算 null 结尾的字符串长度的函数。

因为我们目前还没有写过打印任何数据到标准输出的代码，所以目前只能通过将计算结果以 exit code 的形式返回，并在 exit 的系统调用完成之前写入寄存器。如果想要知道上一个退出程序的退出代码，可以使用 $? 变量来获取。

```
> true
> echo $?
0
> false
> echo $?
1
```

我们来写一个模仿 false shell 命令的汇编程序，代码参考列表 2-13。

_**Listing 2-13**.false.asm_

```
global _start

section .text
_start:
    mov rdi, 1
    mov rax, 60
    syscall
```

现在我们有了需要计算字符串长度的一切工具了。列表 2-14 展示了这样的功能代码。

_**Listing 2-14**.String Length:strlen.asm_

```
global _start

section .data

test_string: db "abcdef", 0

section .text

strlen:                   ; by our convention, first and the only argument
                          ; is taken from rdi
    xor rax, rax          ; rax will hold string length. If it is not
                          ; zeroed first, its value will be totally random

.loop:                    ; main loop starts here
    cmp byte [rdi+rax], 0 ; Check if the current symbol is null-terminator.
                          ; We absolutely need that 'byte' modifier since
                          ; the left and the right part of cmp should be
                          ; of the same size. Right operand is immediate
                          ; and holds no information about its size,
                          ; hence we don't know how many bytes should be
                          ; taken from memory and compared to zero
    je .end               ; Jump if we found null-terminator
    inc rax               ; Otherwise go to next symbol and increase
                          ; counter
    jmp .loop

.end:
    ret                   ; When we hit 'ret', rax should hold return value

_start:

    mov rdi, test_string
    call strlen
    mov rdi, rax

    mov rax, 60
    syscall
```

重要的 strlen 函数我们暂时先留着。请注意：

1. strlen 修改了寄存器的值，所有调用 strlen 之后寄存器的值就发生变化了。
2. strlen 没有修改 rbx 或者其它 callee-saved 寄存器。

---

**■Question 19** 你能指出列表 2-15 中的一到两个 bug 么？什么情况下会触发?

---

_**Listing 2-15**. strlen 的另一个版本：strlen\_bug1.asm_

```
global _start

section .data
test_string: db "abcdef", 0

section .text

strlen:
.loop:
    cmp byte [rdi+r13], 0
    je .end
    inc r13
    jmp .loop
.end:
    mov rax, r13
ret

_start:
    mov rdi, test_string
    call strlen
    mov rdi, rax
    mov rax, 60
    syscall
```




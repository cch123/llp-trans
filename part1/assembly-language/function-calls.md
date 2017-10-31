2.4 函数调用

过程或者叫函数允许你把一段程序逻辑独立出来然后将其作为黑盒使用，这也是为了能够提供必要的抽象。抽象可以使你在将复杂算法封装为不透明的接口，然后用来构建更为复杂的系统。

`call <address>` 指令用来调用函数。这条指令行为如下：

```
push rip
jmp <address>
```

执行之后保存在栈中的地址\(也就是之前 rip 寄存器中的内容\)，叫作**返回地址**。

任何一个函数都可以接收无穷大小个参数。这些参数中前六个参数分别通过 rdi，rsi，rdx，rcx，r8 和 r9 进行传递。剩下的参数则会以相反的顺序被推到栈上进行传入。

我们还没法明确地表示一个函数过程的结束。最直观的方式可以认为 ret 指令标识函数的终结。ret 指令的语义和 pop rip 是完全一致的。

显然，call 和 ret 的调用策略非常脆弱，必需要有其它方式来保证栈的状态能够小心操作。在栈恢复到调用当前函数之前的状态之前，不应该调用 ret 指令。否则的话处理器会将当前栈顶元素作为返回地址而使用其作为 rip 的内容，显然可能会去执行垃圾指令。

现在来探讨一下函数应该怎么样使用寄存器。执行函数会修改寄存器，这是显而易见的。按照保存方式，可把寄存器分为两种类型：

* **callee-saved 寄存器**：这些寄存器必须由被调用的过程来恢复。所以函数如果需要修改这些寄存器的值，那就需要在使用完毕之后把它们恢复回来。rbx，rbp，rsp，r12-r15 这七个寄存器都是需要被调用方恢复的寄存器。
* **caller-saved 寄存器**：这些寄存器需要在调用其它函数之前保存，并在调用完毕之后恢复。不过如果这些寄存器的值在调用函数之后没什么用了的话，那也可以不用保存和恢复。
  除了上一条提到的七个寄存器，其余的都是 caller-saved 寄存器。

上面的两种类型是一种规范/约定。也就是说程序员必须遵守这样的协议：

* 保存并恢复 callee-saved 寄存器。
* 时刻注意 caller-saved 寄存器在调用函数的过程中可能会被修改。

---

**■A source of bugs** bug 源，在调用函数之前不保存 caller-saved 寄存器，并且在函数返回之后又使用这些寄存器是一种常见的错误。请记住：

如果你修改了 rbx，rbp，rsp，r12-r15，把它们恢复回来！

如果你需要任何寄存器在函数调用发生之后还有效，请自己保存它们的值！

---

一些函数有**返回值**。这个返回值经常代表着函数本身的目的和执行的结果。例如我们写一个函数，输入一个数字，返回数字的平方。

从实现习惯上来讲，我们会把结果存储在 rax 这个寄存器中。如果你需要返回两个值怎么办？另一个值可以存储在 rdx 中。

这样调用函数的模式就是下面这样：

* 保存所有的 caller-saved 寄存器\(可以使用 push 来完成\)
* 把参数放到对应的寄存器
* 使用 call 指令调用函数
* 函数返回后，rax 里有函数的返回值
* 恢复 caller-saved 寄存器为调用函数之前的状态

---

**■Why do we need conventions?** 函数是用来抽象一段逻辑，使我们可以忘掉它内部的实现细节并在需要的时候可以修改它，同时使得修改对外部完全透明。前面提到的规范可以使你在任何地方调用任何函数而且能保证结果没有问题\(会修改 caller-saved 寄存器，但一定会保证 callee-saved 寄存器不变\)。

---

---

**■On system call arguments** 系统调用的参数使用的寄存器和普通函数调用使用的寄存器稍有不同，第四个参数被存在了 r10 中，而函数调用第四个参数存在 rcx！

---

原因在于 syscall 指令会隐式地使用 rcx。系统调用无法接收六个以上的参数。

如果你不遵守上面提到的规范，你就没有办法在调用函数的时候避免可能引起的 bug。

现在是时候来再写两个函数了：\`print\_newline\` 会打印换行符；\`print\_hex\` 会接收一个数字打印其 16 进制形式 \(参考列表 2-10\)。

_**Listing 2-10**.print\_call.asm_

```
section .data

newline_char: db 10
codes: db '0123456789abcdef'

section .text
global _start

print_newline:
    mov rax, 1            ; 'write' syscall identifier
    mov rdi, 1            ; stdout file descriptor
    mov rsi, newline_char ; where do we take data from
    mov rdx, 1            ; the amount of bytes to write
    syscall
   ret

print_hex:
    mov rax, rdi

    mov rdi, 1
    mov rdx, 1
    mov rcx, 64           ; how far are we shifting rax?
iterate:
    push rax              ; Save the initial rax value
    sub rcx, 4
    sar rax, cl           ; shift to 60, 56, 52, ... 4, 0
                          ; the cl register is the smallest part of rcx
    and rax, 0xf          ; clear all bits but the lowest four
    lea rsi, [codes + rax]; take a hexadecimal digit character code

    mov rax, 1            ;
    push rcx              ; syscall will break rcx
    syscall               ; rax = 1 (31) -- the write identifier,
                          ;   rdi = 1 for stdout,
                          ; rsi = the address of a character, see line 29

    pop rcx
    pop rax               ; ˆ see line 24 ˆ
    test rcx, rcx         ; rcx = 0 when all digits are shown
    jnz iterate

    ret

_start:
    mov rdi, 0x1122334455667788
    call print_hex
    call print_newline

    mov rax, 60
    xor rdi, rdi
    syscall
```




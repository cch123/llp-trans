14.3.1 Volatile and setjmp

编译器认为 setjmp 只是一个函数。然而 setjmp 并不真的是一个函数，其实际上是一个程序能够开始执行的“点”。一般情况下，在 setjmp 执行之前，一些局部变量被缓存在寄存器中\(或未被分配内存\)。当我们从 longjmp 调用返回到这个"点"时，这些变量不会被恢复。

关闭优化会改变这种行为。所以关闭优化可能会隐藏使用 setjmp 所导致的 bug。

想要正确完成程序的话，记住只有 volatile 类型的局部变量在 longjmp 之后还持有正确的值。这些变量并不像一般的变量一样会被恢复为旧的值，因为 jmp buf 并不保存栈上的变量，而是保存那些 longjmp 之前的变量值。

列表 14-15 展示了一个示例。

_**Listing 14-15**.setjmp\_volatile.c_

```
#include <stdio.h>
#include <setjmp.h>

jmp_buf buf;

int main( int argc, char** argv ) {
    int var = 0;
    volatile int b = 0;
    setjmp( buf );
    if (b < 3) {
        b++;
        var ++;
        printf( "\n\n%d\n", var );
        longjmp( buf, 1 );
    }
    return 0; 
}
```

我们使用 gcc -O0\(列表 14-16\) 和 gcc -O2\(列表 14-17\)级别优化分别编译这个程序

Listing 14-16.volatile\_setjmp\_o0.asm

    main:
    push     rbp
    mov      rbp,rsp
    sub      rsp,0x20

    ; `argc` and `argv` are saved in stack to make `rdi` and `rsi` available
    mov    DWORD PTR [rbp-0x14],edi
    mov    QWORD PTR [rbp-0x20],rsi

    ; var = 0
    mov    DWORD PTR [rbp-0x4],0x0

    ;b= 0
    mov DWORD PTR [rbp-0x8],0x0

    ; 0x600a40 is the address of `buf` (a global variable of type `jmp_buf`)
    mov    edi,0x600a40
    call   400470 <_setjmp@plt>

    ; if (b < 3), the good branch is executed
    ; This is encoded by skipping several instructions to the `.endlabel` if b > 2
    mov    eax,DWORD PTR [rbp-0x8]
    cmp    eax,0x2
    jg     .endlabel

    ; A fair increment
    ; b++
    mov    eax,DWORD PTR [rbp-0x8]
    add    eax,0x1
    mov    DWORD PTR [rbp-0x8],eax

    ; var++
    add    DWORD PTR [rbp-0x4],0x1

    ; `printf` call
    mov    eax,DWORD PTR [rbp-0x4]
    mov    esi,eax
    mov    edi,0x400684
    ; There are no floating point arguments, thus rax = 0
    mov    eax,0x0
    call   400450 <printf@plt>

    ; calling `longjmp`
    mov    esi,0x1
    mov    edi,0x600a40
    call   400490 <longjmp@plt>

    .endlabel:
    Mov    eax,0x0
    Leave
    ret

在开启优化的情况下，程序输出是：

1

2

3

_**Listing 14-17**.volatile\_setjmp\_o2.asm_

    main:

    ; allocating memory in stack
    sub    rsp,0x18

    ; a `setjmp` argument, the address of `buf`

    mov    edi,0x600a40

    ;b= 0
    mov DWORD PTR [rsp+0xc],0x0
    ; instructions are placed in the order different
    ; from C statements to make better use of pipeline and other inner ; CPU mechanisms.
    call 400470 <_setjmp@plt>

    ; `b` is read and checked in a fair way
    mov    eax,DWORD PTR [rsp+0xc]
    cmp    eax,0x2
    jle    .branch

    ; return 0
    xor    eax,eax
    add    rsp,0x18
    ret

    .branch:

    mov    eax,DWORD PTR [rsp+0xc]

    ; the second argument of `printf` is var + 1
    ; It was not even read from memory nor allocated.
    ; The computations were performed in compile time
    mov    esi,0x1

    ; The first argument of `printf`
    mov    edi,0x400674

    ;b=b+ 1
    add eax,0x1
    mov DWORD PTR [rsp+0xc],eax

    xor eax,eax
    call   400450 <printf@plt>

    ; longjmp( buf, 1 )
    mov    esi,0x1
    mov    edi,0x600a40
    call   400490 <longjmp@plt>

程序的输出将是

1

1

1

volatile 变量 b，行为和预期相符\(否则，循环就永远结束不了了\)。即使你在程序中对其进行 increment 操作，变量 var 始终还是等于 1。

---

■Question 265 如何用 setjmp 和 longjmp 实现类似 “try-catch” 式的结构呢？

---




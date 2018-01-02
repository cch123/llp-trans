16.1.7 返回值优化

拷贝的避免和返回值的优化使我们能够消除不必要的拷贝操作。

局部变量往往是在函数的栈上被创建。所以如果一个函数返回了一个结构体类型的实例，函数需要在栈上先创建这个变量，然后将变量拷贝到外面的世界\(除非变量可以被存储到 rax 和 rdx 两个寄存器上\)。

列表 16-14 展示了一个例子。

_**Listing 16-14**. nrvo.c_

```
struct p  {
    long x;
    long y;
    long z;
};

__attribute__ ((noinline))
    struct p f(void) {
        struct p copy;
        copy.x = 1;
        copy.y = 2;
        copy.z = 3;
        return copy;
}

int main(int argc, char** argv) {
    volatile struct p inst = f();
    return 0;
}
```

一个名叫 copy 的 struct p 的实例在栈帧上创建并离开。它的字段被赋值为 1，2 和 3，然后被拷贝到外部，我们推测可能是被 f 作为一个隐藏的参数所接收了。

列表 16-15 是结果的汇编代码。

    Listing 16-15. nrvo_off.asm
    00000000004004b6 <f>:
    ; prologue
    4004b6:  55                             push   rbp
    4004b7:  48 89 e5                       mov    rbp,rsp
    ; A hidden argument is the address of a structure which will hold the  result.
    ; It is saved into stack.
    4004ba:  48 89 7d d8                    mov    QWORD PTR [rbp-0x28],rdi
    ; Filling the fields of `copy` local variable
    4004be:  48 c7 45 e0 01 00 00
    4004c5:  00
    4004c6:  48 c7 45 e8 02 00 00
    4004cd:  00
    mov    QWORD PTR [rbp-0x20],0x1
    mov    QWORD PTR [rbp-0x18],0x2
    mov    QWORD PTR [rbp-0x10],0x3
    4004ce:  48 c7 45 f0 03 00 00
    4004d5:  00
    ; rax = address of the destination struct

    4004d6:  48 8b 45 d8                   mov    rax,QWORD PTR [rbp-0x28]
    ; [rax] = 1 (taken from `copy.x`)
    4004da:  48 8b 55 e0                   mov    rdx,QWORD PTR [rbp-0x20]
    4004de:  48 89 10                      mov    QWORD PTR [rax],rdx
    ; [rax + 8] = 2 (taken from `copy.y`)
    4004da:  48 8b 55 e0                   mov    rdx,QWORD PTR [rbp-0x20]
    4004e1:  48 8b 55 e8                   mov    rdx,QWORD PTR [rbp-0x18]
    4004e5:  48 89 50 08                   mov    QWORD PTR [rax+0x8],rdx
    ; [rax + 10] = 3 (taken from `copy.z`)
    4004e9:  48 8b 55 f0                   mov    rdx,QWORD PTR [rbp-0x10]
    4004ed:  48 89 50 10                   mov    QWORD PTR [rax+0x10],rdx
    ; rax =  address where we have put the structure contents
    ; (it was the hidden argument)
    4004f1:  48 8b 45 d8                   mov    rax,QWORD PTR [rbp-0x28]
    4004f5:  5d                            pop    rbp
    4004f6:  c3                            ret

    00000000004004f7 <main>:
    4004f7:  55                             push   rbp
    4004f8:  48 89 e5                       mov    rbp,rsp
    4004fb:  48 83 ec 30                    sub    rsp,0x30
    4004ff:  89 7d dc                       mov    DWORD PTR [rbp-0x24],edi
    400502:  48 89 75 d0                    mov    QWORD PTR [rbp-0x30],rsi
    400506:  48 8d 45 e0                    lea    rax,[rbp-0x20]
    40050a:  48 89 c7                       mov    rdi,rax
    40050d:  e8 a4 ff ff ff                 call   4004b6 <f>
    400512:  b8 00 00 00 00                 mov    eax,0x0
    400517:  c9                             leave
    400518:  c3                             ret
    400519:  0f 1f 80 00 00 00 00           nop    DWORD PTR [rax+0x0]

编译器能够产生更高效的代码，如列表 16-16 所示。

_**Listing 16-16**.nrvo\_on.asm_

```
00000000004004b6 <f>:
4004b6:  48 89 f8                       mov     rax,rdi
4004b9:  48 c7 07 01 00 00 00           mov     QWORD PTR [rdi],0x1
4004c0:  48 c7 47 08 02 00 00           mov     QWORD PTR [rdi+0x8],0x2
4004c7:  00
4004c8:  48 c7 47 10 03 00 00           mov QWORD PTR [rdi+0x10],0x3
4004cf:  00
4004d0:  c3                             ret

00000000004004d1 <main>:
4004d1:  48 83 ec 20                    sub     rsp,0x20
4004d5:  48 89 e7                       mov     rdi,rsp
4004d8:  e8 d9 ff ff ff                 call    4004b6 <f>
4004dd:  b8 00 00 00 00                 mov     eax,0x0
4004e2:  48 83 c4 20                    add     rsp,0x20
4004e6:  c3                             ret
4004e7:  66 0f 1f 84 00 00 00           nop     WORD PTR [rax+rax*1+0x0]
4004ee:  00 00
```

我们没有为了拷贝而在栈帧上分配任何空间！而是直接操作结构体并将其作为一个隐藏参数传给我们。

如何活用本节结论？如果你想写一个函数来填充某个结构体，传入预分配好内存空间的指针并没有什么收益。


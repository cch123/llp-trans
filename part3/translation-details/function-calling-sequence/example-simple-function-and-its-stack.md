14.1.3 示例：简单的函数和调用栈

我们来看一个简单的函数，功能为计算两个值的最大值。在没有进行优化的情况下编译它，查看输出的汇编：

列表 14-4 展示了这个例子：

Listing 14-4.maximum.c

```
int maximum( int a, int b ) {
    char buffer[4096];
    if (a < b) return b;
    return a;
}

int main(void) {
    int x = maximum( 42, 999 );
    return 0;
}
```

列表 14-5 展示了使用 objdump 进行反汇编的输出内容。

_**Listing 14-5**.maximum.asm_

```
0000000000400546 <maximum>:
  400546:    55                       push   rbp
  400547:    48 89 e5                 mov    rbp,rsp
  40054a:    48 81 ec 20 10 00 00     sub    rsp,0x1020
  400551:    89 bd ec ef ff ff        mov    DWORD PTR [rbp-0x1014],edi
  400557:    89 b5 e8 ef ff ff        mov    DWORD PTR [rbp-0x1018],esi
  40055d:    64 48 8b 04 25 28 00     mov    rax,QWORD PTR fs:0x28
  400564:    00 00
  400566:    48 89 45 f8              mov    QWORD PTR [rbp-0x8],rax
  40056a:    31 c0                    xor    eax,eax
  40056c:    8b 85 ec ef ff ff        mov    eax,DWORD PTR [rbp-0x1014]
  400572:    3b 85 e8 ef ff ff        cmp    eax,DWORD PTR [rbp-0x1018]
  400578:    7d 08                    jge    400582 <maximum+0x3c>
  40057a:    8b 85 e8 ef ff ff        mov    eax,DWORD PTR [rbp-0x1018]
  400580:    eb 06                    jmp    400588 <maximum+0x42>
  400582:    8b 85 ec ef ff ff        mov    eax,DWORD PTR [rbp-0x1014]
  400588:    48 8b 55 f8              mov    rdx,QWORD PTR [rbp-0x8]
  40058c:    64 48 33 14 25 28 00     xor    rdx,QWORD PTR fs:0x28
  400593:    00 00
  400595:    74 05                    je     40059c <maximum+0x56>
  400597:    e8 84 fe ff ff           call   400420 <__stack_chk_fail@plt>
  40059c:    c9                       leave
  40059d:    c3                       ret

000000000040059e <main>:
  40059e:    55                       push   rbp
  40059f:    48 89 e5                 mov    rbp,rsp
  4005a2:    48 83 ec 10              sub    rsp,0x10
  4005a6:    be e7 03 00 00           mov    esi,0x3e7
  4005ab:    bf 2a 00 00 00           mov    edi,0x2a
  4005b0:    e8 91 ff ff ff           call   400546 <maximum>
  4005b5:    89 45 fc                 mov    DWORD PTR [rbp-0x4],eax
```

我们把代码精简为一些可读的汇编代码，如 14-6 所示。

Listing 14-6.maximum\_refined.asm

```
mov rsi, 999
mov rdi, 42
call maximum

...
maximum:
push rbp
mov rbp, rsp
sub rsp, 3984

mov [rbp-0x1004], edi
mov [rbp-0x1008], esi
mov eax, [rbp-0x1004]
...

Leave
ret
```

---

**■Register assignment** 参考 3.4.2 节，来获知为什么修改 esi 寄存器就相当于修改整个 rsi 寄存器。

---

我们将会跟踪函数调用以及调用前的准备工作 \(见列表 14-6\)，并且展示在执行之后马上查看栈中的内容。

call maximum

![](/assets/14-a.gif)

push rbp

![](/assets/14-b.gif)

mov rbp, rsp

![](/assets/14-c.gif)

sub rsp, 3984

![](/assets/14-d.gif)


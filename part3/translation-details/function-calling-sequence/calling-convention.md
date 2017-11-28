14.1.2 调用规约

**调用规约**是程序员需要遵守的关于函数调用顺序的约定。

如果所有人都遵守同样的规则的话，那么就可以流畅地完成合作。一旦有人破坏规则，例如，修改或者在一个函数中不恢复 rbp 的值，那么可能会发生：啥事都没有，稍后程序崩溃，或者程序马上就崩。由于其它函数在编写的时候都假定自身以外的函数是遵守这些调用规则的，且保证 rbp 寄存器不被修改。

调用规约定义的其中之一就是参数的传递算法。我们这里使用传统的 \*nix 的 x86 64 下的习惯\(在\[24\]中详细定义\)，下面的描述对于函数如何被调用是一个相对精确的近似。

1. 首先，需要对值进行保护的寄存器需要保存好原来的值。除了七个 callee-saved 寄存器\(rbx，rbp，rsp，r12 - r15\)都可能被被调用函数所修改，所以如果这些寄存器的值重要的话，就需要把这些寄存器的值保存起来\(一般都在栈上保存\)。
2. 寄存器和栈都会被参数填充。
   每个参数都会被 round 到 8 字节。
   参数会被分为三个列表：
   \(a\) 整数和指针参数
   \(b\) Float 和 double 参数
   \(c\) 通过内存中的栈传入的参数

第一个列表中的前六个参数通过六个通用寄存器传入\(rdi，rsi，rdx，rcx，r8 和 r9\)。第二个列表中的前八个参数通过 xmm0 ~ xmm7 这八个寄存器传入。如果前两个列表还有更多的参数传入，那么这些多余的参数会被以**反序**存储在栈上传入。也就是说，在函数被执行前，传入的最后一个参数应该是在栈顶上。

While integers and floats are quite trivial to handle, structures are a bit trickier.

If a structure is bigger than 32 bytes, or has unaligned fields, it is passed in memory.

A smaller structure is decomposed in fields and each field is treated separately and, if in an inner structure, recursively. So, a structure of two elements can be passed the same way as two arguments. If one field of a structure is considered “memory,” it propagates to the structure itself.

Therbpregister, as we will see, is used to address the arguments passed in memory and local variables.

What about return values? Integer and pointer values are returned inraxandrdx. Floating point values are returned inxmm0andxmm1. Big structures are returned through a pointer, provided as an additional hidden argument, in the spirit of the following example:

```
struct s {

    char vals[100];
};

struct s f( int x ) {
    struct s mys;
    mys.vals[10] = 42;
    return mys;
}

void f( int x, struct s* ret ) {
    ret->vals[10] = 42;
}
```

3.Then thecallinstruction should be called. Its parameter is the address of the first

instruction of a called function. It pushes the return address into the stack.

Each program can have multiple instances of the same function launched at the same time, not only in different threads but also due to recursion. Each such function instance is stored in the stack, because its main principle—“last in, first out”—corresponds to how functions are launched and terminated. If a functionfis launched and then invokes a functiong,gis terminated first \(but was invoked last\), andfis terminated last \(while being invoked first\).

Stack frameis a part of a stack dedicated to a single function instance. It stores the values of the local variables, temporal variables, and saved registers.

The function code is usually enclosed inside a pair ofprologueandepilogue, which are similar for all functions. Prologue helps initialize the stack frame, and epilogue deinitializes it.

During the function execution,rbpstays unchanged and points to the beginning of its stack frame. It is possible to address local variables and stack arguments relatively torbp. It is reflected in the function prologue shown in Listing14-1.

_**Listing 14-1**.prologue.asm_

```
func:
push rbp
mov rbp, rsp

sub rsp, 24      ; given 24 is total size of local variables
```

The oldrbpvalue is saved to be restored later in epilogue. Then a newrbpis set up to the current top of the stack \(which stores the oldrbpvalue now by the way\). Then the memory for the local variables is allocated in the stack by subtracting their total size fromrsp. This is the automatic memory allocation in C and the technique we have used in the very first assignment to allocate buffers on stack.

The functions end with an epilogue shown in Listing14-2.

Listing 14-2.epilogue.asm

```
mov rsp, rbp
pop rbp
ret
```

By moving the stack frame the beginning address intorspwe can be sure that all memory allocated in the stack is deallocated. Then the oldrbpvalue is restored, and nowrbppoints at the start of the previous stack frame. Finally,retpops the return address from stack intorip.

A fully equivalent alternative form is sometimes chosen by the compiler. It is shown in Listing14-3.

Listing 14-3.epilogue\_alt.asm

```
Leave
ret
```

The leave instruction is made especially for stack frame destruction. Its counterpart,enter, is not always used by compilers because it is more functional than the instruction sequence shown in Listing14-1. It is aimed at languages with inner functions support.

4.After leaving the function, our work is not always done. In case there were arguments that were passed in memory \(stack\), we have to get rid of them too.


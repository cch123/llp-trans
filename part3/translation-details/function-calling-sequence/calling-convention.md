14.1.2 调用规约

**调用规约**是程序员需要遵守的关于函数调用顺序的约定。

如果所有人都遵守同样的规则的话，那么就可以流畅地完成合作。一旦有人破坏规则，例如，修改或者在一个函数中不恢复 rbp 的值，那么可能会发生：啥事都没有，稍后程序崩溃，或者程序马上就崩。由于其它函数在编写的时候都假定自身以外的函数是遵守这些调用规则，并保证 rbp 寄存器不被修改的。

 If everyone is following the same rules, a smooth interoperability is guaranteed. However, once someone breaks the rules, for example, makes changes, and does not restorerbpin a certain function, anything can happen: nothing, a delayed crash, or an immediate one. The reason is that other functions are written with the implication that these rules are respected and they count onrbpbeing left untouched.

The calling conventions declare, among other things, the argument passing algorithm. In the case of the typical \*nix x86 64 convention we are using \(described fully in \[24\]\), the description that follows is an accurate enough approximation of how the function is called.

1.First, the registers that need to be preserved are saved. All registers except for seven callee-saved registers \(rbx,rbp,rsp, andr12-r15\) can be changed by the called function, so if their value is of any importance, they should be stored \(probably in a stack\).

2.The

The size of each argument gets rounded up to 8 bytes. The arguments are split into three lists:

1. \(a\) Integer or pointer arguments.

2. \(b\) Floats and doubles.

3. \(c\) Arguments passed in memory via stack \(“memory”\).

The first six arguments from the first list are passed in general purpose registers \(rdi,rsi,rdx,rcx,r8, andr9\). The first eight arguments from the second list are passed in registersxmm0toxmm7. If there are more arguments from these lists to pass, they are passed on to the stack in reverse order. It means that the last argument will be on top of the stack before the call is performed.

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


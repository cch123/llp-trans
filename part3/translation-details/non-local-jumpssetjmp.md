14.3 Non-Local jumps–setjmp

C 标准库包含了可以做 tricky 的 hack 的一些手段。这些手段允许保存计算的上下文并在之后进行恢复。上下文描述了程序执行的状态，除了下面这些特殊内容：

* 和外部世界打交道的任意对象\(e.g.打开的文件描述符\)
* 浮点数计算上下文
* 栈上的变量

除此之外的内容，C 标准库的工具允许你保存当前上下文并在我们想返回的时候跳回到这些上下文。这种跳转并不会被函数的作用域所限制。

把 setjmp.h 引入到你的工程中就可以使用上面说的工具了：

* jmp\_buf 是一种用来存储上下文的变量类型。
* int setjmp\(jmp\_buf env\) 是一个函数，该函数接收一个 `jmp buf`实例并把当前的上下文保存在这个传入的 jmp buf 中。默认情况下该函数会返回 0。
* void longjmp\(jmp\_buf env, int val\) 函数用来恢复保存的上下文，传入的 jmp buf 即“保存的上下文”。

从 longjmp 中返回时，setjmp 会返回传给 longjmp 的 val 值，而非 0。列表 14-13 展示了一个例子。第一个 setjmp 默认返回 0，并将 0 赋值给 val。然而 longjmp 接收 1 作为其参数，然后程序会从 longjmp 跳转到 setjmp 调用\(因为这两个调用之间使用 jb 变量串联起来了\)。这次 setjmp 则会返回 1，而这个 1 这次也会赋值给 val。

_**Listing 14-13**.longjmp.c_

```
#include <stdio.h>
#include <setjmp.h>

int main(void) {
    jmp_buf jb;
    int val;
    val = setjmp( jb );
    puts("Hello!");
    if (val == 0) longjmp( jb, 1 );
    else puts("End");
    return 0;
}
```

Local variables that are not marked volatile will all hold undefined values afterlongjmp. This is the source of bugs as well as memory freeing related issues: it is hard to analyze the control flow in presence oflongjmpand ensure that all dynamically allocated memory is freed.

In general, it is allowed to callsetjmpas a part of a complex expression, but only in rare cases. In most cases, this is an undefined behavior. So, better not to do it.

It is important to remember that all this machinery is based on stack frames usage. It means that you cannot performlongjmpin a function with a deinitialized stack frame. For example, the code, shown in Listing14-14, yields an undefined behavior for this very reason.

Listing 14-14.longjmp\_ub.c

```
jmp_buf jb;
void f(void) {
    setjmp( jb );
}

void g(void) {
    f();
    longjmp(jb);
}
```

The functionfhas terminated already, but we are performing longjmp into it. The program behavior is undefined because we are trying to restore a context inside a destroyed stack frame.

In other words, you can only jump into the same function or into a function that is launched.


14.3 非局部跳转–setjmp

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

没有被标记为 volatile 类型的局部变量在 longjmp 之后会持有一个未定义的值。这也是一种 bug 的源头，这些 bug 的表现和那些内存释放方面的 bug 比较类似，而且 longjmp/setjmp 导致的 bug 难以分析，因为程序的控制流被修改掉了，而且这种情况下也难以保证动态申请的内存都正确地被释放掉了。

如果遇到了复杂的表达式，那 setjmp 调用还是可以适当放行的，不过这也是说的很少见的情况下。大多数情况下，这是一种未定义的行为，所以最好不要碰。

请记住，这里讲到的手段都是基于栈帧的使用的。也就是说你没法在栈帧已经被销毁的函数执行 longjmp。例如列表 14-14，就引起了这里说的未定义行为。

_**Listing 14-14**.longjmp\_ub.c_

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

函数 f 已经被销毁掉了，然而我们却用 longjmp 想要跳进 f 函数中。这种行为是未定义的，因为我们想要恢复存储在已经销毁的栈帧的上下文。

换句话说，你只能跳到同一个函数的其它位置，或者已经在执行的函数中\(已压栈\)。


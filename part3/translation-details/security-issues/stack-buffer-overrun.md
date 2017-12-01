14.7.1 栈缓冲溢出

假设程序使用了一个函数 f 和本地 buffer，如列表 14-25 所示：

_**Listing 14-25**.buffer\_overrun.c_

```
#include <stdio.h>

void f( void ) {
    char buffer[16];
    gets( buffer );
}

int main( int argc, char** argv ) {
    f();
    return 0;
}
```

初始化之后，栈帧的结构看起来会像下面这样：

![](/assets/14-g.png)

gets 函数从 stdin 中读取一行内容并把其放进在参数给定了地址的 buffer 里。不幸的是，函数没有控制 buffer 的大小，从而可以越过 buffer。

如果该行内容太长的话，就可能会覆盖掉 buffer 内容，甚至可能会覆盖掉保存在 buffer 上 rbp 的值，或者函数的返回地址。接下来执行指令的话就很可能让程序整个 crash。更糟糕的情况，如果攻击者更精心地准备了一行输入，可能把返回地址改成一个合法的特殊地址。

万一攻击者选择将返回地址重定向到 buffer 之外，他就可以把任何可执行的代码传送到这个 buffer 中了。这种代码一般被称为 shellcode，因为 shellcode 一般都比较小而且会开启一个远程的 shell 来搞破坏。

显然这并不只是 gets 这个函数的瑕疵，而是语言本身的特性。所以永远都不要直接使用 gets，应该提供检查目标内存边界的方法。


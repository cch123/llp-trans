14.8.1 Security Cookie

Security cookie\(stack guard, canary\) 机制，通过在返回地址被修改时强制终止程序，来保护程序不执行任意外部代码。

Security cookie 本身是一个在栈帧上 rbp 和返回地址附近的随机值。

![](/assets/14-f.gif)

越过 buffer 会覆盖掉 security cookie 的内容。在 ret 指令之前，编译器会对 security cookie 进行一次特殊的检查，如果其被修改的话，就直接让程序崩溃。ret 指令就不会被执行了。

MSVC 和 GCC 都实现了这种机制，并且默认是打开的。


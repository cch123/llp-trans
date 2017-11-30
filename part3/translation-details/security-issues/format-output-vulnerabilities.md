14.7.3 格式化输出的弱点

输出格式化函数是一些恶心 bug 的源头。标准库里有一些这种格式化函数；表格 14-1 列出了这些函数。

Table 14-1.String Format Functions

| Function | Description |
| :--- | :--- |
| printf | 输出格式化后的字符串 |
| fprintf | 将 printf 的结果写入到文件 |
| sprintf | 打印到一个字符串 |
| snprintf | 打印到字符串的同时检查长度 |
| vfprintf | 把 va\_arg 结构打印到文件 |
| vprintf | 把 va\_arg 打印到 stdout |
| vsprintf | 把 va\_arg 打印到字符串 |
| vsnprintf | 把 va\_arg 打印到字符串同时检查长度 |

列表 14-26 展示了一个例子。假设用户输入了少于 100 个符号。你能让这个程序挂掉或者产生其它有意思的效果么？

_**Listing 14-26**.printf\_vuln.c_

```
#include <stdio.h>
int main(void) {
    char buffer[1024];
    gets(buffer);
    printf( buffer );
    return 0;
}
```

这种弱点并不是因为使用了 gets 函数，而是由于对用户输入的格式化而导致的。用户的输入中可能会包含格式化占位符，从而导致有趣的行为。我们会提到一些潜在的意外行为。

"%x" 占位符和其类似的符号能够用来查看栈内容。前五个 "%x" 将从寄存器器中取参数\(rdi 已经被 format 字符串地址占据了\)，紧跟着的几个会展示出栈内容来。我们把列表 14-26 中的例子编译一下，看看对这个程序输入 "%x %x %x %x %x %x %x %x %x %x %x" 会发生什么情况。

```
> %x %x %x %x %x %x %x %x %x %x
b1b6701d b19467b0 fbad2088 b1b6701e 0 25207825 20782520 78252078 25207825
```

正如我们所见，输出结果是四个有些共同点的数值，一个 0 然后和两个其它的数值。按照我们的假设，最后两个数字已经是从栈上获取的了。

进入 gdb 来探索一下 printf 调用之后的栈顶内容，我们可以得到证明观点的证据。列表 14-27 展示了 gdb 中的输出。

_**Listing 14-27**.gdb\_printf_

```
(gdb) x/10 $rsp
0x7fffffffdfe0: 0x25207825 0x78252078   0x20782520 0x25207825
0x7fffffffdff0: 0x78252078 0x20782520   0x25207825 0x00000078
0x7fffffffe000: 0x00000000 0x00000000
```

* •The"%s" format specifier is used to print strings. As a string is defined by the address of its start, this means addressing memory by a pointer. So, if no valid pointer is given, the invalid pointer will be dereferenced.

---

■Question 266 What will be the result of launching the code shown in listing14-26on input"%s %s %s %s %s"?

---

* •The"%n"format specifier is a bit exotic but still harmful. It allows one to write an integer into memory. Theprintffunction accepts apointerto an integer which will be rewritten with an amount of symbols written so far \(before"%n"occurs\). Listing14-28shows an example of its usage.

_**Listing 14-28**.printf\_n.c_

```
#include <stdio.h>

int main(void) {
    int count;
    printf( "hello%n world\n", &count);
    printf( "%d\n", count );
    return 0;
}
```

这个程序会输出 5，因为在 "%n" 之前有五个符号输出。这个并不是一个莫名其妙的字符串长度，因为在前面可能还有其它的格式化占位符，而这些占位符可能影响到变量的长度输出\(e.g.打印一个 integer 可能会产生七或十个符号\)。列表 14-29 展示了一个例子、

_**Listing 14-29**.printf\_n\_ex.c_

```
int x;
printf("%d %n", 10, &x);  /* x = 3 */
printf("%d %n", 200, &x); /* x = 4 */
```

To avoid that, do not use the string accepted from the user as a format string. You can always writeprintf\("%s", buffer\), which is safe as long as the buffer is not NULL and is a valid null-terminated string. Do not forget about such functions asputsoffputs, which are not only faster but also safer.


14.6 Strict Aliasing

restrict 被引入之前，程序员有时使用不同的结构体名字来获取到相同的效果。编译器看到数据类型不同进而推断这些指针不能指向相同的数据\(这个规则一般被称为 strict aliasing 严格别名规则\)。

这种规则存在这样一些假设：

* 指向不同内置类型的指针不会是别名。
* 指向有不同 tag 的结构体或 union 结构的指针不会互为别名\(所以 struct foo 和 struct bar 不会混用并指向对方\)。
* 使用 typedef 创建的类型别名可能指向相同的数据。
* char \* 类型\(signed 或者 unsigned\)是一个特例。编译器始终假设 char\* 可以是其它类型的别名，但是反过来不成立。也就是说我们可以创建一个 char buffer，使用这个 char buffer 来获取一些数据，然后将它作为一个结构体的实例的别名。

破坏这些规则就可能会引起难查的优化 bug，因为这样会触发未定义行为。

列表 14-18 中的例子，可以在不使用 restrict 关键字的前提下获得与 restrict 关键字一样的效果。也就是使用 strict aliasing 来达到我们的目的，把两个参数打包到具有不同的 tag 的 struct 中去。

列表 14-23 展示了修改后的源码。

_**Listing 14-23**.restrict-hack.c_

```
struct a {
    int v;
};

struct b {
    int v;
};

void f(struct a* x, struct b* add) {
    x->v += add->v;
    x->v += add->v;
}
```

结果令我们很满意，编译器把读取优化掉了。列表 14-24 展示了反编译的结果。

_**Listing 14-24**.restrict-hack-dump_

```
0000000000000000 <f>:
   0:   8b 06                     mov   eax,DWORD PTR [rsi]
   2:   01 c0                     add   eax,eax
   4:   01 07                     add   DWORD PTR [rdi],eax
   6:   c3                        ret
```

我们不鼓励在 C99 或更新的标准下写的代码使用上述 aliasing 规则，因为 restrict 显然更加一目了然，并且也不会引入没什么必要的类型名。


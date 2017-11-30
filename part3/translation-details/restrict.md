14.5 restrict

restrict 是一个类似于 volatile 和 const 的关键字，其首次出现是在 C99 标准中。用来标记指针，语法是是放在星号的右边，像下面这样：

```
int x;
int* restrict p_x = &x;
```

如果我们创建了指向某个对象的 restricted 指针，那么我们就承诺所有针对这个对象的修改都会经过这个指针。编译器会看情况来决定到底是忽略这个 restrict 还是让其发挥作用来实现一些优化，当然这种优化经常是可行的。

换句话说，任何通过其它指针进行的写入都不会影响到 restricted 指针存储的值。如果破坏了这个承诺便会导致诡异的 bug 和未定义的行为。

如果没有 restrict 的话，指针就只是一段内存的别名，你可以用不同的名字访问相同的内存。考虑一下列表 14-18 这个简单的例子。函数 f 的函数体和 \*x += 2\*\(\*add\); 这个表达式是等价的吗？

_**Listing 14-18**. restrict\_motiv.c_

```
void f(int* x, int* add) {
    *x += *add;
    *x += *add;
}
```

答案可能会让你非常意外，两者是不等价的。如果 x 和 add 指向的是相同的内存地址的话会怎么样？这种情况下修改 \*x 也会导致 \*add 被修改。所以函数 f 实际上会让 \*x 最终变成四倍的原始值。即使是 x != add 且 \*x == \*add 时，最终的结果也是原始值的三倍。

编译器对于这些边界 case 非常熟悉，即使在开启优化的情况下也不会把两次读取的操作优化掉，参见列表 14-19。

_**Listing 14-19**.restrict\_motiv\_dump.asm_

```
0000000000000000 <f>:
0:   8b 06               mov   eax,DWORD PTR [rsi]
2:   03 07               add   eax,DWORD PTR [rdi]
4:   89 07               mov   DWORD PTR [rdi],eax
6:   03 06               add   eax,DWORD PTR [rsi]
8:   89 07               mov   DWORD PTR [rdi],eax
a:   c3                  ret
```

不过我们只要像 14-20 这样加上 restrict 修饰，反编译的结果就不一样了，见 14-21。第二个参数只被读取了一次，然后乘以 2，再加到第一个参数的解引用结果上。

_**Listing 14-20**.restrict\_motiv1.c_

```
void f(int* restrict x, int* restrict add) {
    *x += *add;
    *x += *add;
}
```

_**Listing 14-21**.restrict\_motiv\_dump1.asm_

```
0000000000000000 <f>:
   0:   8b 06              mov   eax,DWORD PTR [rsi]
   2:   01 c0              add   eax,eax
   4:   01 07              add   DWORD PTR [rdi],eax
   6:   c3                 ret
```

只有在你确定自己在做什么的时候再去用 restrict。毕竟写一个效率不太高的程序总比写一个错的程序要好。

用 restrict 来为文档化代码也很重要。例如 memcpy 这个函数的签名，memcpy 是一个从起始地址拷贝 n 字节到 目标地址的函数，该函数在 C99 中有变化：

```
void* memcpy(void*       restrict s1,
       const void* restrict s2,

       size_t              n );
```

这反映出两块区域是没有交叉的；否则的话正确性就没法保证了。

restrict 修饰的指针可以被拷贝从而创建出多层的指针。然而标准限制使得拷贝目标指针和原始始终要在同一块作用域。列表 14-22 展示了这样的示例。

_**Listing 14-22**.restrict\_hierarchy.c_

```
struct s {
    int* x;
} inst;

void f(void) {
    struct s* restrict p_s = &inst;
    int* restrict p_x = p_s->x; /* Bad */
    {
        int* restrict p_x2 = p_s->x; /* Fine, other block scope */
    }
}
```




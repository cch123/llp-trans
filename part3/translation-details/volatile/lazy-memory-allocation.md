14.2.1 内存懒分配

很多操作系统对页的映射都采用“懒映射原则”，实际的映射时机时在调用 mmap \(或类似的映射函数\) 之后第一次使用该内存时再加载\(而非调用函数的时候\)。

如果程序员不希望系统做这种“首次使用映射”的事情，那么可以选择对每个页分别进行一次寻址，这样操作系统就真的会创建该页，像列表 14-9 中所列的一样。

_**Listing 14-9**.lma\_bad.c_

```
char* ptr;
for( ptr = start; ptr < start + size; ptr += pagesize )
*ptr;
```

不过这段代码对于编译器来说存在着无法观测到运行结果的问题，可能会被编译器直接优化\(删除\)掉。然而你可以把这个指针标记为 volatile 来避免变量被优化掉。列表 14-10 展示了这样的例子：

_**Listing 14-10**.lma\_good.c_

```
volatile char* ptr;
for( ptr = start; ptr < start + size; ptr += pagesize )

*ptr;
```

---

**■Volatile pointers in the language standard** 如果 volatile 指针指向一段非 volatile 的内存的话，根据语言标准这种情况下没有任何保证！只有在内存和指针都是 volatile 的情况下才符号 volatile 指针的标准，所以，根据标准来看，上面的例子是不正确的。不过因为程序员们这里使用 volatile 指针是为了避免优化的原因，大多数编译器\(MSVC，GCC，Clang\) 也就不会优化掉这些经过解引用的 volatile 指针，这件事情并没有什么符合标准的做法。

---




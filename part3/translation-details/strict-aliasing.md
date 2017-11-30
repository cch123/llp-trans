14.6 Strict Aliasing

restrict 被引入之前，程序员有时使用不同的结构体名字来获取到相同的效果。编译器看到数据类型不同进而推断这些指针不能指向相同的数据\(这个规则一般被称为 strict aliasing 严格别名规则\)。

这种规则存在这样一些假设：

* 指向不同内置类型的指针不会是别名。
* 指向有不同 tag 的结构体或 union 结构的指针不会互为别名\(所以 struct foo 和 struct bar 不会混用并指向对方\)。
* 使用 typedef 创建的类型别名可能指向相同的数据。
* char \* 类型\(signed 或者 unsigned\)是一个特例。编译器始终假设 char\* 可以是其它类型的别名，但是反过来不成立。也就是说我们可以创建一个 char buffer，使用这个 char buffer 来获取一些数据，然后将它作为一个结构体的实例的别名。

破坏这些规则就可能会引起难查的优化 bug，因为这样会触发未定义行为。



The example shown in Listing14-18, can be rewritten to achieve the same effect without therestrictkeyword. The idea is to use the strict aliasing rules to our benefit, packing both parameters into the structures with different tags.

Listing14-23shows the modified source.

Listing 14-23.restrict-hack.c

codecodecode

To our satisfaction, the compiler optimizes the reads away just as we wanted. Listing14-24shows the disassembly.

Listing 14-24.restrict-hack-dump

codecodecode

We discourage using aliasing rules for optimization purposes in code for C99 and newer standards becauserestrictmakes the intention more obvious and does not introduce unnecessary type names.


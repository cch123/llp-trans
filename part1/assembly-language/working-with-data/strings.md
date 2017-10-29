2.5.2 Strings

我们已经了解到字符串是使用 ASCII 表来进行编码的。一个编码可以赋值给任何一个字符。一个字符串显然就是一个字符编码的序列。不过这些描述中没有说明如何判断字符串的长度。

1. 显式指定字符串长度的：
   db 27, 'Selling England by the Pound'
2. 用一个特殊字符标识字符串结尾的。按照传统会使用 0 作为这个“特殊字符”。这种字符叫做 null-terminated 字符串。
   db 'Selling England by the Pound', 0




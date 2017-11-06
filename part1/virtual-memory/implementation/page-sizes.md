4.7.3 Page Sizes

不同层级的表的结构是非常相似的。页大小可能会被调整为 4KB，2MB 或者 1GB。依赖于表的结构，这种层级可以收缩至最少两个级别。这种情况下 PDP 会像页表一样工作，并且会存储 1GB 帧的一部分。参考图 4-4 来查看表条目的格式如何随着页大小的变化而变化。

这里有两张图

Figure 4-4.Page Directory Pointer table and Page Directory table entry format

这种行为是被第 7 位对应的 PDP 或者 PD 条目来控制的。如果设置了该位的话，对应的表会映射页；否则的话，它会存储下一级表的地址。


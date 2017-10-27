实模式是最古老的模式。这种模式没有虚拟内存的概念；操作系统直接在物理内存上进行寻址，通用寄存器只有 16 位长。

因此，没有什么 rax，eax，只有 ax，al 和 ah。

这些寄存器可以保存从 0 到 65535 大小的值，所以我们使用这样的寄存器也就只有 65535 字节的寻址能力。这样的内存区域被称为段\(segment\)。不要把这里的段和 ELF\(Executable and Linkable Format\)文件中的段\(section\)搞混了。

下面是在实模式中使用到的寄存器：

* ip，flags；
* ax，bx，cx，dx，sp，bp，si，di；
* 段寄存器：cs，ds，ss，es，\(还有 gs 和 fs\)

因为 16 位的寄存器不能简单地定位大于 64 KB 的内存，工程师们想出了一个办法来解决这个问题。使用一种特殊的段寄存器，像下面这样：

* 物理内存地址由 20 位组成\(5 个十六进制数\)

* 每个逻辑地址都由两部分组成。一部分从段寄存器中取，用来标识段的起始位置。另一部分记录的是地址在段内的偏移量。硬件用这两个寄存器来计算物理地址： 

  物理地址 = 段基址  \* 16 + 偏移量

  你可能经常看到以段地址加偏移量来表示的地址，例如：4a40:0002， ds:0001，7bd3:ah。





As we already stated, programmers want to separate code from data \(and stack\), so they intend to use different segments for these code sections. Segment registers are specialized for that:csstores the code segment start address,dscorresponds to data segment, andssto stack segment. Other segment registers are used to store additional data segments.

Note that strictly speaking, the segment registers do not hold segments’ starting addresses but rather their parts \(the four most significant hexadecimal digits\). By adding another zero digit to multiply it by 1610we get the real segment starting address.

Each instruction referencing memory implicitly assumes usage of one of segment registers. Documentation clarifies the default segment registers for each instruction. However, common sense can help as well. For instance,movis used to manipulate data, so the address is relative to the data segment.


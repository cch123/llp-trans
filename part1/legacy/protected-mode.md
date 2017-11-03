3.2 Protected Mode

Intel 80386 是第一个实现了 32 位保护模式的处理器。

它提供了更广泛的寄存器\(eax，ebx，...，esi，edi\) 和新的保护策略：protection rings，虚拟内存，和改善的分段。

这些策略实现了程序之间的彼此隔离，所以程序异常结束不会再影响到其它程序。更进一步，程序没有办法破坏掉其所在处理器的内存了。

相比实模式，获得段的起始地址的方式也改变了。现在起始地址是根据一个特别的表的条目计算得到的，而不是直接通过段寄存器的直接乘法得到的。

```
                   线性地址 = 段基址 \(从系统表中读取\) + 偏移量
```

段寄存器 cs，ds，ss，es，gs，fs 中存储的内容叫做段选择器，寄存器中存储的是一个指向特殊段描述符表的索引和一点附加信息。“段描述符表”分为两种类型：可能有数不清的 LDT\(Local Descriptor Table\) 和一个 GDT\(Global Descriptor Table\)。

LDT 们是为了给硬件的任务切换策略设计的；然而操作系统制造商们没有对 LDT 进行适配。今时今日程序是使用虚拟内存进行隔离的，LDT 没有被使用。

GDTR 是一个寄存器，存储了 GDT 的地址和大小。

段选择器的结构如图 3-1 所示：

![](/assets/3-1.gif)

_**图 3-1**.段选择器\(任意一个段寄存器的内容\)_

Index 表示描述符在 GDT 或者 LDT 中的位置。T 位表示到底是 LDT 还是 GDT。因为 LDT 没人用了，所以无论啥时候，T 这一位始终都是 0。

GDT/LDT 表中的条目还会存储段被赋予了哪种特权级别。当通过段选择器来访问一个段时，会进行一次请求特权级别\(在选择器=段寄存器中存储\) 和描述符特权级别\(在描述符表中存储\)的检查。如果请求特权级别不够访问高特权级别的段，那么就会发生错误。这样我们就可以创建无数权限不一样的段，并用存储在段选择器中的请求特权级别值来定义哪些对我们来说是可访问的了\(在给出我们自己的特权级别的情况下\)。

特权级别和 protection ring 是一回事。

比较稳妥的一种说法是当前的特权级别\(e.g.当前的 ring\)存储在 cs 和 ss 寄存器的低两个 bit\(这两个数值应该是相等的\)。这个值会影响到我们执行特定关键指令的能力\(e.g. 修改 GDT 自身\)。

ds 寄存器的使用也可以很容易地验证这一点，只消修改几个 bit 使我们能覆盖当前的数据访问的特权级别为更低的级别，甚至只能访问一个选定的段。

例如当前我们在 ring-0 模式下，且 ds = 0x02。即使 cs 和 ss 最低的两位都是 0\(因为我们在 ring-0 下\)，我们依然不能访问特权级别比 2 高的段\(例如 1 或者 0 模式下\)。

换句话说，当我们访问段时，请求的特权级别字段保存了我们到底有多大的权限。段会被依次赋予四种 protection ring 的其中一种。当请求访问一个特定的特权级别时，请求的特权级别必须比段的特权级别属性要高。

---

**■Note** 你无法直接修改 cs 寄存器

---

图 3-2 展示了 GDT 描述符的格式。

![](/assets/3-2.gif)

_**图 3-2**.段描述符 \(在 GDT 或 LDT 内\)_

G—Granularity, e.g., size is in 0 = bytes, 1 = pages of size 4096 bytes each. D—Default operand size \(0 = 16 bit, 1 = 32 bit\).  
L—Is it a 64-bit mode segment?  
V—Available for use by system software.

P—Present in memory right now.  
 S—Is it data/code \(1\) or is it just some system information holder \(0\).  
 X—Data \(0\) or code \(1\).  
 RW—For data segment, is writing allowed? \(reading is always allowed\); for code segment, is reading

allowed? \(writing is always prohibited\).  
 DC—Growth direction: to lower or to higher addresses? \(for data segment\); can it be executed from

higher privilege levels? \(if code segment\)  
 A—Was it accessed?  
 DPL—Descriptor Privilege Level \(to which ring is it attached?\)

处理器总是\(即使是今天\)从实模式启动的。为了进入保护模式，需要创建 GDT 然后设置好 gdtr，在 cr0 设置一个特殊的标识位，然后执行一个叫做 far jump 的操作。Far jump 意思是段\(或者段选择器\)已经显式地给出\(并且可以和默认的不同\)，像下面这样：

```
jmp 0x08:addr
```

列表 3-1 展示了我们如何开启保护模式的代码片断\(假设 start32 是 32 位代码启动时的一个 label\)。

_**Listing 3-1.**开启保护模式 loader\_start32.asm_

```
lgdt cs:[_gdtr]

mov eax, cr0                  ; !! Privileged instruction
or al, 1                      ; this is the bit responsible for protected mode
mov cr0, eax                  ; !! Privileged instruction

    jmp (0x1 << 3):start32    ; assign first seg selector to cs

align 16
_gdtr:                        ; stores GDT's last entry index + GDT address
dw 47
dq _gdt

align 16

_gdt:
; Null descriptor (should be present in any GDT)
dd 0x00, 0x00
; x32 code descriptor:
db 0xFF, 0xFF, 0x00, 0x00, 0x00, 0x9A, 0xCF,     0x00 ; differ by exec bit
; x32 data descriptor:
db 0xFF, 0xFF, 0x00, 0x00, 0x00, 0x92, 0xCF,     0x00 ; execution off (0x92)
;  size  size  base  base  base  util  util|size  base
```

Align 伪指令控制对齐，本书稍后会对它进行说明。

---

■Question 45 解析一下这个段选择器：0x08

---

You might think that every memory transaction needs another one now to read GDT contents. This is not true: for each segment register there is a so-called **shadow register**, which cannot be directly referenced. It serves as a cache for GDT contents. It means that once a segment selector is changed, the corresponding shadow register is loaded with the corresponding descriptor from GDT. Now this register will serve as a source of all information needed about this segment.

D flag 标记需要一点说明，因为它依赖于段的类型。

* It is a code segment: default address and operand sizes. One means 32-bit addresses and 32-bit or 8-bit operands; zero corresponds to 16-bit addresses and 16-bit or 8-bit operands. We are talking about encoding of machine instructions here. This behavior can be altered by preceding an instruction by a prefix0x66\(to alter operand size\) or0x67\(to alter address size\).

* Stack segment \(it is a data segment AND we are talking about one selected by ss\).2It is again default operand size forcall,ret,push/pop, etc. If the flag is set, operands are 32-bit wide and instructions affectesp; otherwise operands are 16-bit wide andspis affected.

* For data segments, growing toward low addresses, it denotes their limits \(0 for 64 KB, 1 for 4 GB\). This bit should always be set in long mode.

就像你看到的一样，分段实在是麻烦的玩艺儿。所以没有被操作系统和程序员广泛接受也是有原因的\(现在也基本上被抛弃了\)。

对于程序员来说，不分段更为简单。

广泛使用的程序语言都没有在内存模型上对分段进行支持。而是平整的一块内存。所以安排段的分配是编译器的工作了\(实际上实现起来也比较难\)。

程序分段会造成内存碎片的灾难。

描述符表最多可以保存 8192 个描述符。我们怎么能有效地使用这么小丁点的配额啊。

* After the introduction of long mode segmentation was purged from processor, but not completely. It is still used for protection rings and thus a programmer should understand it.




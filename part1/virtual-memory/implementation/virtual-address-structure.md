4.7.1 虚拟地址结构

每一个 64 位的虚拟地址\(e.g.就是在我们程序内使用的地址\)都由一些字段组成，像图 4-1 中展示的一样。

_**Figure 4-1**.虚拟地址结构_

The address itself is in fact only 48 bits wide; it is sign-extended to a 64-bit **canonical address**. Its characteristic is that its 17 left bits are equal. If the condition is not satisfied, the address gets rejected immediately when used.

Then 48 bits of virtual address are transformed into 52 bits of physical address with the help of special tables.3

---

■Bus Error 当你使用非权威地址时你将会看到一条错误信息：Bus error

---

Physical address space is divided into slots to be filled with virtual pages. These slots are called **page frames**. There are no gaps between them, so they always start from an address ending with 12 zero bits.

The least significant 12 bits of virtual address and of physical page correspond to the address offset inside page, so they are equal.

The other four parts of virtual address represent indexes in translation tables. Each table occupies exactly 4KB to fill an entire memory page. Each record is 64 bits wide; it stores a part of the next table’s starting address as well as some service flags.


3.3 Minimal Segmentation in Long Mode

即使在长模式下每次选择一条指令，处理也会使用到分段。系统提供给我们一维线性虚拟地址，然后再将虚拟地址转换为物理地址\(参考 4.2 节\)。

LDT 是没有被任何人接受的上下文切换策略；正因为此，在长模式中 LDT 被完全抛弃了。

所有通过主要的段寄存器\(cs，ds，es，ss\)来进行寻址都不再考虑 GDT 中的 base 和 offset 了。无论 descriptor 内容是什么，段的起始地址始终是从 0x0 开始；段的大小没有任何限制。除此以外其它的 descriptor 字段不会被忽略。

这么说的意思是，在长模式下 _GDT 中还至少需要三个描述符_：null descriptor\(GDT 中这个字段一定要有\)，代码和数据段。如果你要用 protection ring 来实现特权模式和用户模式，你还需要代码和数据描述符来为用户级代码服务。

---

 **■Why do we need separate descriptors for code and data?** 没有描述符 flag 组合能允许程序员能同时设置读/写和执行权限。

---

现在即使我们在汇编上只有微不足道的经验， 对我们来说破译一段展示典型GDT 的 loader 的代码片段也不是什么难事了。这段代码摘自 Pure64，一个开源的操作系统 loader。由于其是在操作系统启动之前执行的，所以不包括用户级的代码和数据描述符\(见列表 3-2\)。

_**Listing 3-2**.A Sample GDT gdt64.asm_

    align 16  ; This ensures that the next command or data element is
    ; stored starting at an address divisible by 16 (even if we need
    ; to skip some bytes to achieve that).

    ; The following will be copied to GDTR via LGDTR instruction:

    GDTR64:                 ; Global Descriptors Table Register
        dw gdt64_end - gdt64 - 1 ; limit of GDT (size minus one)
        dq 0x0000000000001000    ; linear address of GDT

    ; This structure is copied to 0x0000000000001000
    gdt64:
    SYS64_NULL_SEL equ $-gdt64      ; Null Segment
        dq 0x0000000000000000
    ; Code segment, read/exec, nonconforming
    SYS64_CODE_SEL equ $-gdt64
        dq 0x0020980000000000       ; 0x00209A0000000000
    ; Data segment, read/write, expand down
    SYS64_DATA_SEL equ $-gdt64
        dq 0x0000900000000000       ; 0x0020920000000000
    gdt64_end:

    ; Dollar sign denotes the current memory address, so
    ; $-gdt64 means an offset from `gdt64` label in bytes




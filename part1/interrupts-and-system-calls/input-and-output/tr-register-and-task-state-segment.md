6.1.1 TR register and Task State Segment

There are some artifacts from the protected mode that are still somehow used in long mode. A segmentation is an example, now mostly used to implement protection rings. Another is a pair of atrregister and **Task State Segment **control structure.

The tr register holds the segment selector to the TSS descriptor. The latter resides in the GDT \(Global Descriptor Table\) and has a format similar to segment descriptors.

Likewise for segment registers, there is a shadow register, which is updated fromGDTwhentris updated vialtr\(load task register\) instruction.

The TSS is a memory region used to hold information about a task in the presence of a hardware task-switching mechanism. Since no popular OS has used it in protected mode, this mechanism was removed from long mode. However, TSS in long mode is still used, albeit with a completely different structure and purpose.

These days there is only one TSS used by an operating system, with the structure described in Figure6-1.

_**Figure 6-1**.长模式下的 Task State 段_

The first 16 bits store an offset to an Input/Output Port Permission Map, which we already discussed in section 6.1. The TSS then holds eight pointers to specialinterrupt stack tables\(ISTs\) and stack pointers for different rings. Each time a privilege level changes, the stack is automatically changed accordingly. Usually, the newrspvalue will be taken from the TSS field corresponding to the new protection ring. The meaning of ISTs is explained in section 6.2.


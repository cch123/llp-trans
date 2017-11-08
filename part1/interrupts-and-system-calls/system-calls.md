6.3 System Calls

System calls are, as you already know, functions that an OS provides for user applications. This section describes the mechanism that allows their secure execution with higher privilege level.

The mechanisms used to implement system calls vary in different architectures. Overall, any instruction resulting in an interrupt will do, for example, division by zero or any incorrectly encoded instruction.  
 The interrupt handler will be called and then the CPU will handle the rest. In protected mode on Intel architecture, the interrupt with code 0x80 was used by \*nix operating systems. Each time a user executedint 0x80, the interrupt handler checked the register contents for system call number and arguments.

System calls are quite frequent, and you cannot perform any interaction with the outside world without them. Interrupts, however, can be slow, especially in Intel 64, since they require memory accesses toIDT.

So in Intel 64 there is a new mechanism to perform system calls, which usessyscallandsysretinstructions to implement them.

Compared to interrupts, this mechanism has some key differences:

* The transition can only happen between ring0 and ring3.As pretty much no one uses ring1 and ring2, this limitation is not considered important.

* Interrupt handlers differ, but all system calls are handled by the same code with only one entry point.

* Some general purpose registers are now implicitly used during system call.

  * – rcxis used to store oldrip

  * – r11is used to store oldrflags




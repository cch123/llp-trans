16.1.9 Influence of Execution Units

A CPU consists of many parts. Each instruction is executed in multiple stages, and at each stage different parts of the CPU are handling it. For example, the first stage is usually called instruction fetch and consists of loading instruction from memory1without thinking about its semantics at all.

A part of the CPU that performs the operations and calculations is called theexecution unit. It is implementing different kinds of operations that the CPU wants to handle: instruction fetching, arithmetic, address translation, instruction decoding, etc. In fact, CPUs can use it in a more or less independent  
 way. Different instructions are executed in a different number of stages, and each of these stages can be performed by a different execution unit. It allows for interesting circuitry usages such as the following:

* Fetching one instruction immediately after the other was fetched \(but has not completed its execution\).

* Performing multiple arithmetic actions simultaneously despite their being described sequentially in assembly code.

CPUs of the Pentium IV family were already capable of executing four arithmetic instructions simultaneously in the right circumstances.

How do we use the knowledge about execution unit’s existence? Let us look at the example shown in Listing16-17.

Listing 16-17.cycle\_nonpar\_arith.asm

```
looper:
    mov      rax,[rsi]
    ; The next instruction depends on the previous one.
    ; It means that we can not swap them because
    ; the program behavior will change.
    xor     rax, 0x1

    ; One more dependency here
    add     [rdi],rax
    add     rsi,8
    add     rdi,8
    dec     rcx
    jnz     looper
```

Can we make it faster? We see the dependencies between instructions, which hinder the CPU microcode optimizer. What we are going to do is to unroll the loop so that two iterations of the old loop become one iteration of the new one. Listing16-18shows the result.

Listing 16-18.cycle\_par\_arith.asm

```
looper:
mov      rax,  [rsi]
mov      rdx,  [rsi + 8]
xor      rax,  0x1
xor      rdx,  0x1
add      [rdi], rax
add      [rdi+8], rdx
add      rsi, 16

add      rdi, 16
sub      rcx, 2
jnz      looper
```

Now the dependencies are gone, the instructions of two iterations are now mixed. They will be executed faster in this order because it enhances the simultaneous usage of different CPU execution units. Dependent instructions should be placed away from each other to allow other instructions to perform in between.

---

■Question 333 What is the instruction pipeline and superscalar architecture?

---

We cannot tell you which execution units are in your CPU, because this is highly model dependent. We have to read the optimization manuals for a specific CPU, such as \[16\]. Additional sources are often helpful; for example, the Haswell processors are well explained in \[17\].


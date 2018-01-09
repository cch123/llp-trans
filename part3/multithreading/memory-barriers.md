17.7 Memory Barriers

Memory barrieris a special instruction or statement that imposes constraints on how the reorderings can be done. As we have seen in the Chapter16, compilers or hardware can use many tricks to improve performance in the average case, including reordering, deferred memory operations, speculative loads or branch prediction, caching variables in registers, etc. Controlling them is vital to ensure that certain operations are already performed, because the other thread’s logic depends on it.

In this section, we want to introduce the different kinds of memory barriers and give us a general idea about their possible implementations on Intel 64.

An example of the memory barrier preventing reorderingsby compileris the following GCC directive:

asm volatile\("" ::: "memory"\)

Theasmdirective is used to include inline assembly code directly into C programs. Thevolatilekeyword together with the"memory"clobber argument describes that this \(empty\) piece of inline assembly cannot be optimized away or moved around and that it performs memory reads and/or writes. Because of that, the compiler is forced to emit the code to commit all operations to memory \(e.g., store the values of the local variables, cached in registers\). It does not prevent the processor from performing speculative reads past this statement, so it isnota memory barrier for the processor itself.

Obviously, both compiler and CPU memory barriers arecostlybecause they prevent optimizations. That is why we do not want to use them after each instruction.

There are several kinds of memory barriers. We will speak about those that are defined in the Linux kernel documentation, but this classification is applicable in most situations.

1.Write memory barrier.  
 It guarantees that allstoreoperations specified in code before the barrier will

appear to happen before allstoreoperations specified after the barrier.

GCC usesasm volatile\(""::: "memory"\)as a general memory barrier. Intel 64 uses the instructionsfence.

2.Readmemorybarrier.

Similarly, it guarantees that allloadoperations specified in code before the barrier will appear to happen before allloadoperations specified after the barrier. It is a stronger form of data dependency barrier.

GCC usesasm volatile\(""::: "memory"\)as a general memory barrier. Intel 64 uses the instructionlfence.

3.Datadependencybarriers.

Data dependency barrier considers the dependent reads, described in section 17.4. It can be thus considered a weaker form of read memory barrier. No guarantees about independent loads or any kinds of stores are provided.

4.Generalmemorybarriers

This is the ultimate barrier, which forces every memory change specified in code before it is committed. It also prevents all following operations to be reordered in a way they appear to be executed before the barrier.

GCC usesasm volatile\(""::: "memory"\)as a general memory barrier. Intel 64 uses the instructionmfence.

5.Acquireoperations.

This is a class of operations, united by a property calledAcquire semantics. If  
 an operation performsreadsfrom shared memory and is guaranteed to not be reordered with thefollowing reads and writesin the source code, it is said to have this property.

In other words, it is similar to a general memory barrier, but the code that follows will not be reordered in a way to be executed before this barrier.

6.Release operations.

Release semanticsis a property of such operations. If an operation performswritesto shared memory and is guaranteed to not be reordered with theprevious reads and writesin the source code, it is said to have this property.

In other words, it is similar to a general memory barrier but stillallowsthe more recent operations to be reordered in a position before the release operation.

Acquire and release operations, thus, are one-way barriers for reorderings in a way. Following is an example of a single assembly commandmfence, inlined by GCC:

asm \("mfence" \)

By combining it with the compiler barrier, we get a line that both prevents compiler reordering and also acts as a full memory barrier.

```
asm volatile("mfence" ::: "memory")
```

Any function callwhose definition is not available in the current translation unitand that isnot an intrinsic\(a cross-platform substitute of a specific assembly instruction\) is acompiler memory barrier.




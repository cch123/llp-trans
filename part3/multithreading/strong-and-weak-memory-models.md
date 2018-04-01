17.4 强弱内存模型

编译器在编译时可能对程序进行内存重排\(类似之前展示过的\)，处理器在微代码级别也会进行内存重排。一般情况下这两种内存重排都是会发生的，两种我们都会进行研究。

我们可以以统一的方式对其进行分类。

内存模型会告诉我们 load 和 store 指令会以哪种形式重排。这里我们不关心访问访问内存的具体指令\(mov, movq, etc\)，只关心对内存到底是读操作，还是写操作。



There are two extreme poles: weak and strong memory models. Just like with the strong and weak typing, most existing conventions fall somewhere between, closer to one or another. We have found a classification made by Jeff Preshing \[31\] to be useful and will stick to it in this book.

According to it, the memory models can be divided into four categories, enumerated from the more relaxed ones to the stronger ones.

Really weak.  
 In these models, any kind of memory reordering can happen \(as long as the

1. observable behavior of a single-threaded program is unchanged, of course\).

2. Weakwithdatadependencyordering\(suchashardwarememorymodelof ARM v7\).

   Here we speak about one particular kind of data dependency: the one betweenloads. It occurs when we need to fetch an address from memory and then use it to perform another fetch, for example,

   ```
            Mov rdx, [rbx]
            mov rax, [rdx]
   ```

   In C this is the situation when we use the➤operator to get to a field of a certain structure through the pointer to that structure.

   Really weak memory models do not guarantee data dependency ordering.

3. Usuallystrong\(suchashardwarememorymodelofIntel64\).

   It means that there is a guarantee that allstoresare performed in the same order as provided. Someloads, however, can be moved around.

   Intel 64 is usually falling into this category.

4. Sequentially consistent.

This can be described as what you see when you debug a non-optimized program step by step on a single processor core. No memory reordering ever happens.


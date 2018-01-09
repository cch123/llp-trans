17.4 Strong and Weak Memory Models

Memory reorderings can be performed by the compiler \(as shown above\), or by the processor itself, in the microcode. Usually, both of them are being performed and we will be interested in both of them.  
 A uniform classification can be created for them all.

Amemory modeltells us which kinds of reorderings ofloadandstoreinstructions can be expected. We are not interested in the exact instructions used to access memory most of the time \(mov,movq, etc.\), only the fact of reading or writing to memory is of importance to us.

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




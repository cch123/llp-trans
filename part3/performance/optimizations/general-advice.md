16.1.2 General Advice

When programming we should not usually be concerned with optimizations right away. Premature optimization is evil for numerous reasons.

* Most programs are written in a way that only a fraction of their code is repeatedly executed. This code determines how fast the program will be executing, and it can slow down everything else. Speeding up other parts will in these circumstances have little to no effect.

* Finding such parts of the code is best performed using aprofiler—a utility program that measures how often and how long different parts of code are executed.

* Optimizing code by hand virtually always makes it less readable and harder to maintain.

* Modern compilers are aware of common patterns in high-level language code. Such patterns get optimized well because the compiler writers put much work into it, since the work is worth it.

The most important part of optimizations is often choosing the right algorithm. Low-level optimizations on the assembly level are rarely so beneficial. For example, accessing elements of a linked list by index  
 is slow, because we have to traverse it from the beginning, jumping from node to node. Arrays are more beneficial when the program logic demands accessing its elements by index. However, insertion in a linked list is easy compared to array, because to insert an element to thei-th position in an array we have to move all following elements first \(or maybe even reallocate memory for it first and copy everything\).

A simple, clean code is often also the most efficient.

Then, if the performance is unsatisfactory, we have to locate the code that gets executed the most using profiler and try to optimize it by hand. Check for duplicated computations and try to memorize and reuse computation results. Study the assembly listings and check if forcing inlining for some of the functions used is doing any good.

General concerns about hardware such as locality and cache usage should be taken into account at this time. We will speak about them in section 16.2.

The compiler optimizations should be considered then. We will study the basic ones later in this section. Turning specific optimizations onor offfor a dedicated file or a code region can have a positive impact on performance. By default, they are usually all turned on when compiling with -O3flag.

Only then come lower-level optimizations: manually throwing in SSE or AVX \(Advanced Vector Extensions\) instructions, inlining assembly code, writing data bypassing hardware cache, prefetching data into caches before using it, etc.

The compiler optimizations are boldly controlled by using compiler flags-O0,-O1,-O2,-O3,-Os\(optimize space usage, to produce the smallest executable file possible\). The index near-O, increases as the set of enabled optimizations grows.

Specific optimizations can be turned on and off. Each optimization type has two associated compiler options for that, for example,-fforward-propagateand-fno-forward-propagate.




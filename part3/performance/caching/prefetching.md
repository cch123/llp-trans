16.2.2 预取

It is possible to issue a special hint to the CPU to indicate that a certain memory area will be accessed soon. In Intel 64 it is done using aprefetchinstruction. It accepts an address in memory; the CPU will do its best to preload it into cache in the near future. This is used to prevent cache misses.

Using prefetch can be effective enough, but it should be coupled with testing. It should be  
 executed before the data accesses themselves, but not too close. The cache preloading is being executed asynchronously, which means that it is a running at the same time when the following instructions are being executed. Ifprefetchis too close to data accesses, the CPU will not have enough time to preload data in cache and cache-miss will occur anyway.

Moreover, it is very important to understand that “close” and “far” from the data access mean the instruction position in the execution trace. We should not necessarily placeprefetchclose with regard to the program structure \(in the same function\), but we have to choose a place that precedes data access. It can be located in an entirely different module, for example, in the logging module, which justhappens to usually be executed before the data access. This is of course very bad for code readability, introduces non-obvious dependencies between modules, and is a “last resort” kind of technique.

To use prefetch in C, we can use one of GCC built-ins:

Void \_\_builtin\_prefetch \(const void \*addr, ...\)

It will be replaced with an architecture-specific prefetching instruction.  
 Besides address, it also accepts two parameters, which should be integer constants.

1.Will we read from that address \(0, default\) or write \(1\)?

2.How strong is locality? Three for maximal locality to zero for minimal. Zero indicates that the value can be cleared from cache after usage, 3 means that all levels of caches should continue to hold it.

Prefetching is performed by the CPU itselfifit can predict where the next memory access is likely to be. While it works well for continuous memory accesses, such as traversing arrays, it starts being ineffective as soon as the memory access pattern starts seeming random for the predictor.


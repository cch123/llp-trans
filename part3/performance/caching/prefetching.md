16.2.2 预取

通过给 CPU 发送一个特殊的 hint 来表明之后某段内存区域马上会被访问，是可能的。在 Intel 64 架构上使用 **prefetch** 指令来达到这个目的。该指令接收一个内存地址；CPU 会在最近尽量将这些内容载入到缓存中。这样做可以防止 cache miss。

prefetch 使用得好可以很高效，不过一定要进行测试。prefetch 操作应该在数据被访问前进行，但不能和执行执行的时间间隔太接近。缓存的预载是异步进行的，也就是说数据载入和紧跟着的指令执行几乎是同时发生的。如果预取和数据访问时间太接近的话，CPU 可能来不及把数据载入到 cache，数据访问就发生了。这时候就会有 cache-miss。

另外还需要理解一点，与数据访问操作距离的 “近” 和 "远" 指的是指令执行的序列中的指令位置。在考虑程序的结构前提下

Moreover, it is very important to understand that “close” and “far” from the data access mean the instruction position in the execution trace. We should not necessarily place prefetch close with regard to the program structure \(in the same function\), but we have to choose a place that precedes data access. It can be located in an entirely different module, for example, in the logging module, which justhappens to usually be executed before the data access. This is of course very bad for code readability, introduces non-obvious dependencies between modules, and is a “last resort” kind of technique.

To use prefetch in C, we can use one of GCC built-ins:

Void \_\_builtin\_prefetch \(const void \*addr, ...\)

It will be replaced with an architecture-specific prefetching instruction.  
 Besides address, it also accepts two parameters, which should be integer constants.

1.Will we read from that address \(0, default\) or write \(1\)?

2.How strong is locality? Three for maximal locality to zero for minimal. Zero indicates that the value can be cleared from cache after usage, 3 means that all levels of caches should continue to hold it.

Prefetching is performed by the CPU itselfifit can predict where the next memory access is likely to be. While it works well for continuous memory accesses, such as traversing arrays, it starts being ineffective as soon as the memory access pattern starts seeming random for the predictor.


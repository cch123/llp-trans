17.8.1 什么时候使用多线程

对程序逻辑来说，多线程有时候挺方便的。例如，一般情况下你不应该在同一个线程上进行网络包收发和界面渲染。界面线程需要响应用户行为\(点击按钮\)并经常进行重绘\(例如，当响应的窗口被其它窗口挡住，之后又恢复的时候\)。然而网络行为则会一直其工作线程，直到完成收发工作。这也就是为什么要将这种行为划分到其它线程中去做并发执行。

多线程可以提升程序的性能，不过这种说法不能覆盖所有 case。常见的任务分为 CPU 密集和 I/O 密集型。

* CPU 密集型代码在分配更多 CPU 的情况下就可以更快执行。它会使用大部分的 CPU 时间来进行计算，且在该过程中不从磁盘或外部设备中读取数据。
* IO 密集型代码在分配更多 CPU 时间的情况下也不会被加速，因为这种代码主要是慢在对内存和外部设备的大量使用。

使用多线程来加速 CPU 密集型的程序是可能的。一般的模式会使用一个队列，将请求分发给线程池\(其中有一些预分配的线程，可能在工作，也可能在等待，这样就不用在每次需要的时候都创建新线程\)中的工作线程。参考第 7 章 \[23\] 来获知详情。

As for how many threads we need, there is no universal recipe. Creating threads, switching between them, and scheduling them produces an overhead. It might make the whole program slower if there is not much work for threads to do. In computation-heavy tasks some people advise to spawnn −1 threads, wherenis the total number of processor cores. In tasks that are sequential by their own nature \(where every step depends directly on the previous one\) spawning multiple threads will not help. What we do recommend is toalwaysexperiment with the number of threads under different workloads to find out which number suits the most for the given task.


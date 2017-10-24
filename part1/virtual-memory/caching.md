让我们从一个无处不在的概念开始：缓存。

互联网可以被看作巨大的数据存储设施。你可以访问它任意一部分的数据，但每次你发起数据获取请求的延迟却是不可忽视的。为了使你的浏览体验更为平滑，web 浏览器会对页面和其元素\(图片，css 样式表等等\)进行缓存。通过这种方式来避免一遍又一遍地对同一份数据进行下载。换句话说，浏览器将数据在硬盘或者内存中进行本地存储，以对数据访问进行加速。不过下载整个因特网的内容到本地是不可能的，毕竟本地计算机的存储资源是非常有限的。

硬盘相比内存来说，空间要大很多，但同时访问也慢很多。这也就是为什么一般在把数据加载到内存中时硬盘的任务一般也就完成了的原因。因为主存对于外部存储来说就是一种缓存。

A hard drive is much bigger than RAM but also a great deal slower. This is why all work with data is done after preloading it in RAM. Thus main memory is being used as a cache for data from external storage.

Anyway, a hard drive also has a cache on its own...”

“On CPU crystal there are several levels of data caches \(usually three: L1, L2, L3\). Their size is ”

“much smaller than the size of main memory, but they are much faster too \(the closest level to the CPU is almost as close as registers\). Additionally, CPUs possess at least an instruction cache \(queue storing instructions\) and a Translation Lookaside Buffer to improve virtual memory performance.

Registers are even faster than caches \(and smaller\) so they are a cache on their own.

Why is this situation so pervasive? In information system, which does not need to give strict guarantees about its performance levels, introducing caches often decreases the average access time \(the time between a request and a response\). To make it work we need our old friend locality: in each moment of time we only have a small working set of data.

The virtual memory mechanism allows us, among other things, to use physical memory as a cache for chunks of program code and data.”


对于一个单任务的操作系统来说，任意时刻只会有一个程序在执行，这种情况下从固定的地址开始，把程序的数据直接放在物理内存是明智的做法。其它的组件\(例如设备驱动、库\)也可以在内存以固定的顺序来存放。

然而在多任务友好的系统里就没法这么做了，我们更需要一种框架来支持并行\(或者伪并行\)执行多个程序。这种情况下操作系统需要一种内存管理机制来应对下列挑战：

* 被执行的程序有任意的大小\(程序的大小甚至会超过内存的总大小\)。因此需要提供部分加载功能，以使系统可以只加载即将使用到的程序部分。

* 内存中同时存在多个程序。

  程序还能和响应缓慢的外部设备交互。这些缓慢的交互过程可能会持续成千上万个 cpu 时钟周期，我们希望能够把宝贵的 CPU 资源借给其它程序使用。而程序间的快速切换只有当程序都已经在内存中时才可能实现；否则的话还要花费大量的时间从外部存储中把要切换的程序读取进来。.

* 程序可以存储到内存的任意位置。  
  如果我们实现这个目标的话，就可以在任意的空闲内存位置加载程序了。即使程序内部使用的是绝对地址也没关系。

* In case of absolute addressing, likemov rax, \[0x1010ffba\], all addresses including starting address become fixed: all exact address values are written into machine code.

* Freeing programmers from memory management tasks as much as possible.

  While programming, we do not want to think about how different memory chips on our target architectures can function, what is the amount of physical memory available, etc. Programmers should pay closer attention to program logic instead.

* Effective usage of shared data and code.  
   Whenever several programs want to access the same data or code \(libraries\) files, it

  is a waste to duplicate them in memory for each additional user. Virtual memory usage addresses these challenges.




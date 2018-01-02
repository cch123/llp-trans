16.1.8 分支预测的影响

在微代码级别，CPU 所执行的动作甚至比机器指令粒度更小；这些微代码为了更好地利用 CPU 资源会被重新排序。

分支预测是一种提升程序执行速度的硬件手段。当 CPU 看到一个条件分支指令\(例如 jg\)时，它可以：

* 开始同时执行执行两个分支；或者
* 猜测下一个执行的分支并开始执行该分支。

当计算结果\(e.g. jg \[rax\] 指令的 GF flag\)的跳跃目标没有确定时，这种预测就可能会发生，我们先以“投机”的形式去执行一些代码来避免浪费时间。

分支预测单元在执行误判时，也会直接失败。这种情况下，如果计算已经完成了，CPU 还需要做一些反向的工作，以使在错误的分支执行过的指令效果都能被反转回来。这个过程比较慢且对程序的性能有影响，但误判发生的概率非常小。

具体的分支预测逻辑依赖于 CPU 模型。一般情况下，有两种预测方式：static 和 dynamic。

* 如果 CPU 没有任何关于跳转的信息\(也就是说第一次执行的时候\)，会使用 static 的算法。具体的算法比较简单，可能如下面这样：
  * - 如果这是向前跳转，我们假设这个跳转还会发生。
  *  -- 如果这是向后跳转，我们假设这件事情不会再发生。

  这种预测是有意义的，例如我们用跳转来实现的循环显然很有可能还会再次发生。
* 如果跳转已经在过去发生过，CPU 就使用更为复杂的算法了。例如，我们使用一个 ring buffer，存储跳转是否发生过。换句话说，这个 ring buffer 中存储所有的跳转历史。使用这种手段的话，只要反复使用长度除以 buffer 的长度就是很好的预测方式。

具体的 CPU 模型信息可以从资料\[16\]中找到。不幸的是，大多数 CPU 内部的信息并不对公众所开放。

如何活用本节结论？当使用 if 然后 else 或者 switch，最好在前面部分使用更可能进入的分支。你也可以使用 GCC 伪指令，例如 builtin expect ，该指令被实现为特殊的跳转指令前缀。





The best source of relevant information with regard to the exact CPU model can be found in \[16\]. Unfortunately, most information about the CPU innards is not disclosed to public.

How do we use it?When using if-then-else or switch start with the most likely cases. You can also use special hints such as\_\_builtin\_expectGCC directives, which are implemented as special instruction prefixes for jump instructions \(see \[6\]\).


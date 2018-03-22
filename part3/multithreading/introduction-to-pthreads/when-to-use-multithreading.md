17.8.1 When to Use Multithreading

Sometimes multithreading is convenient for the program logic. For example, you usually should not accept a network packet and draw the graphical interface in the same thread. The graphical interface should react to user actions \(clicks on the buttons\) and be redrawn constantly \(e.g., when the corresponding window gets covered by another window and then uncovered\). The network action, however, will block the working thread until it is done. It is thus convenient to split these actions into different threads to perform them seemingly simultaneously.

Multithreading can naturally improve performance, but not in all cases. There are CPU bound tasks and IO bound tasks.



* CPU bound code can be sped up if given more CPU time. It spends most of the CPU time doing computations, not reading data from disk or communicating with devices.

* IO bound code cannot be sped up with more CPU time because it is slowed by its excessive usage of memory or external devices.

Using multithreading to speed CPU bound programs might be beneficial. A common pattern is to use a queue with the requests that are dispatched to the worker threads from a thread pool–a set of created threads that are either working or waiting for work but are not re-created each time there is a need of them. Refer to Chapter7of \[23\] for more details.

As for how many threads we need, there is no universal recipe. Creating threads, switching between them, and scheduling them produces an overhead. It might make the whole program slower if there is not much work for threads to do. In computation-heavy tasks some people advise to spawnn −1 threads, wherenis the total number of processor cores. In tasks that are sequential by their own nature \(where every step depends directly on the previous one\) spawning multiple threads will not help. What we do recommend is toalwaysexperiment with the number of threads under different workloads to find out which number suits the most for the given task.


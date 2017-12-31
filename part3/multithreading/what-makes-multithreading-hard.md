17.2 What Makes Multithreading Hard?

Multithreading allows you to make use of several processor cores \(or several processors\) to execute threads at the same time. For example, if one thread is reading file from a disk \(which is a very slow operation\), the other might use the pause to perform CPU-heavy computations, distributing CPU \(central processing unit\) load more uniformly in time. So, it can be faster, if your program can benefit from it.

Threads should often work on the same data. As long as the data is not modified by any of them, there are no problems working with it, because reading data has zero effect on other threads execution. However, if the shared data is being modified by one \(or multiple\) threads, we face several problems, such as the following:

* When does threadAsee the changes performed byB?

* In which order do threads change the data? \(As we have seen in the Chapter16, the

  instructions can be reordered for optimization purposes.\)

* How can we perform operations on the complex pieces of data without other threads interfering?



When these problems are not addressed properly, a very problematic sort of bug appears, which is hard to catch \(because it only appears casually, when the instruction of the different threads are executed in a specific, unlucky order\). We will try to establish an understanding and study these problems and how they can be solved.


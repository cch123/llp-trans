17.8.9 Spinlocks

A mutex is a sure way of doing synchronization. Trying to lock a mutex which is already taken by another thread puts the current thread into a sleeping state. Putting the thread to sleep and waking it up has its costs, notably for the context switch, but if the waiting is long, these costs justify themselves. We spend a little time going to sleep and waking up, but in a prolonged sleep state the thread does not use the CPU.

What would be an alternative? The active idle, which is described by the following simple pseudo code:

```
while ( locked == true ) {
    /* do nothing */
}
locked = true;
```



The variablelockedis a flag showing whether some thread took the lock or not. If another thread took the lock, the current thread will constantly poll its value until it is changed back. Otherwise it proceeds to take the lock on its own. This wastes CPU time \(and increases power consumption\), which is bad. However, it can increase performance in case the waiting time is expected to be very short. This mechanism is calledspinlock.

Spinlocks only make sense on multicore and multiprocessor systems. Using spinlock in a single core is useless. Imagine a thread enters the cycle inside the spinlock. It keeps waiting for other thread to change  
 thelockedvalue, but no other thread is executing at this very time, because there is only one core switching from thread to thread. Eventually the scheduler will put the current thread to sleep and allow other threads to perform, but it just means that we have wasted CPU cycles executing an empty loop for no reason at all! In this case, going to sleep right away is always better, and hence there is no use for a spinlock.

This scenario can of course occur on a multicore system as well, but there is also a \(usually\) good chance, that the other thread will unlock the spinlock before the time quantum given to the current thread expires.

Overall, using spinlocks can be beneficial or not; it depends on the system configuration, program logic, and workload. When in doubt, test and prefer using mutexes \(which are often implemented by first taking a spinlock for a number of iterations and then falling into the sleep state if no unlock occurred\).

Implementing afastand correct spinlock in practice is not that trivial. There are questions to be answered, such as the following:

* Do we need a memory barrier on lock and/or unlock? If so, which one? Intel 64, for example, haslfence,sfence, andmfence.

* How do we ensure that the flag modification is atomic? In Intel 64, for example, an instructionxchgsuffices \(withlockprefix in case of multiple processors\).

pthreadsprovide us with a carefully designed and portable mechanism of spinlocks. For more information, refer to themanpages for the following functions:

* pthread\_spin\_lock
* pthread\_spin\_destroy
* pthread\_spin\_unlock




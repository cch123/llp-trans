17.8.6 Deadlocks  


A sole mutex is rarely a cause of problems. However, when you lock multiple mutexes at a time, several kinds

of strange situations can happen. Take a look at the example shown in Listing17-14.

Listing 17-14.deadlock\_ex

```
mutex A, B;
thread1 () {
    lock(A);    
    lock(B);
    unlock(B);
    unlock(A);
}

thread2() {
    lock(B);    
    lock(A);
    unlock(A);
    unlock(B);
}
```

This pseudo code demonstrates a situation where both threads can hang forever. Imagine that the following sequence of actions happened due to unlucky scheduling:

* Thread 1 lockedA; control transferred to thread 2.

* Thread 2 lockedB; control transferred to thread 1. 

After that, the threads will try to do the following:

* Thread 1 will attempt to lockB, butBis already locked by thread 2.

* Thread 2 will attempt to lockA, butAis already locked by thread 1.

Both threads will be stuck in this state forever. When threads are stuck in a locked state waiting for each other to perform unlock, the situation is calleddeadlock.



The cause of the deadlock is the different order in which the locks are being taken by different threads. It leads us to a simple rule that will save us most of the times when we need to lock several mutexes at a time.

---

■Preventing deadlocksOrder all mutexes in your program in an imaginary sequence. Only lock mutexes in

the same order they appear in this sequence.  


For example, suppose we have mutexesA,B,C, andD. We impose a natural order on them:A &lt; B &lt; C &lt; D. if

you need to lock bothDandB, you should always lock them in the same order, thusBfirst,Dsecond. if thisinvariantis kept, no two threads will lock a pair of mutexes in a different order.

---

---

■Question 362What are Coffman’s conditions? how can they be used to diagnose deadlocks?

■Question 363how do we useHelgrindto detect deadlocks?

---




17.13 Summary

In this chapter we have studied the basics of multithreaded programming. We have seen the different memory models and the problems that emerge from the fact compiler and hardware optimizations mess with the instruction execution order. We have learned how to control them, placing different memory barriers, we have seen whyvolatileis not a solution to problems that emerge from multithreading. Then we introducedpthreads, the most common standard of writing multithreaded applications of Unix-like systems. We have seen thread management, used mutexes and condition variables, and learned why spinlocks only have meaning on multicore and multiprocessor systems. We have seen how memory reorderings should be taken into account even when working on an usually strong architecture such as Intel 64 and have seen the limits of its strictness. Finally, we have studied the atomic variables—a very useful feature of C11 that allows us to get rid of explicit mutex usage and in many cases boost performance while maintaining correctness. Mutexes are still important when we want to perform complex manipulations on non-trivial data structures.

---

■Question 368 Which problems emerge from multithreading usage?  
■Question 369 What makes multiple threads worth it?  
■Question 370 Should we use multithreading even if the program does not perform many computations? if yes, give a use case.

■Question 371 What is compiler reordering? Why is it performed?  
■Question 372 Why does the single-threaded program have no means to observe compiler memory reorderings?

■Question 373 What are some kinds of memory models?

■Question 374 how do we write the code that is sequentially consistent with regard to manipulation of two shared variables?

■Question 375 arevolatilevariables sequentially consistent?  
Question 376 Show an example when memory reorderings can lead to very unexpected program behavior.

Question 377 What are the arguments against usage of volatile variables?  
Question 378 What is a memory barrier?  


Question 379 What kinds of memory barriers do you know?  


Question 380 What is acquire semantics?  


Question 381 What is release semantics?  


■Question 382 What is a data dependency? Can you write code where data dependency does not force an order on operations?

■Question 383What is the difference betweenmfence,sfence, andlfence?

■Question 384Why do we need instructions other thanmfence?

■Question 385 Which function calls act as compiler barriers?  


■Question 386 areinlinefunction calls compiler barriers?  


■Question 387 What is a thread?  
■Question 388 What is the difference between threads and processes?

■Question 389 What constitutes the state of a process?

■Question 390 What constitutes the state of a thread?  


■Question 391 Why should the-pthreadflag be used when compiling withpthreads?  
■Question 392 ispthreadsa static or dynamic library?  
■Question 393 how do we know in which order the scheduler will execute the threads?  
■Question 394 Can one thread get access to the stack of the other thread?  


■Question 395 What doespthread\_joindo and how do we use it?  
■Question 396 What is a mutex? Why do we need it?  
■Question 397 Should every shared constant variable be associated with a mutex?  


■Question 398 Should every shared mutable variable which is never changed be associated with a mutex?

■Question 399 Should every shared mutable variable which is changed be associated with a mutex?  
■Question 400 Can we work with a shared variable without ever using a mutex?  


■Question 401What is a deadlock?  
■Question 402 how do we prevent deadlock?  
■Question 403 What is a livelock? how is it different from a deadlock?  
■Question 404 What is a spinlock? how is it different from a livelock and a deadlock?  
■Question 405 Should spinlocks be used on a single core system? Why?  
■Question 406 What is a condition variable?  
■Question 407 Why do we need condition variables if we have mutexes?  
■Question 408 Which guarantees does intel 64 provide for memory reorderings?  
■Question 409 Which important guarantees does intel 64 not provide for memory reorderings?  
■Question 410 Correct the program shown in listing17-18so that no memory reordering occurs.

■Question 411 Correct the program shown in listing17-18so that no memory reordering occurs by using atomic variables.

■Question 412What is lock-free programming? Why is it harder than traditional multithreaded programming with locks?

Question 413What is a CaS operation? how can it be implemented in intel 64?

Question 414 how strong is the C memory model?  


Question 415 Can the strength of the C memory model be controlled?  


Question 416 What is an atomic variable?

Question 417 Can any data type be atomic?  


Question 418 Which atomic variables should be initialized explicitly?

Question 419 Which memory orderings does C11 recognize?

■Question 420 how are the atomic variables manipulation functions with\_explicitsuffix different from their ordinary counterparts?

■Question 421 how do we perform an atomic increment on an atomic variable?  


■Question 422 how do we perform an atomic XOr on an atomic variable?  


■Question 423 What is the difference between weak and strong versions ofcompare\_exchange?

---

  



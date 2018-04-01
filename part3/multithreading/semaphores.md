17.9 Semaphores

Semaphoreis a shared integer variable on which three actions can be performed.

* Initialization with an argumentN. Sets its value toN.
* Wait \(enter\). If the value is not zero, it decrements it. Otherwise waits until someone else increments it, and then proceeds with the decrement.
* Post \(leave\). Increments its value.

Obviously the value of this variable, not directly accessible, cannot fall below 0.

Semaphores arenotpart ofpthreadsspecification; we are working with semaphores whose interface is described in the POSIX standard. However, the code that uses the semaphores should be compiled with a-pthreadflag.

Most UNIX-like operating systems implement both standardpthreadsfeatures and semaphores. Using semaphores is fairly common to perform synchronization between threads.

Listing17-17shows an example of semaphore usage.

Listing 17-17.semaphore\_mwe.c

```
#include <semaphore.h>
#include <inttypes.h>
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>

sem_t sem;

uint64_t counter1 = 0;
uint64_t counter2 = 0;

pthread_t t1, t2, t3;

void* t1_impl( void* _ ) {
    while( counter1 < 10000000 ) counter1++;
    sem_post( &sem );
    return NULL;
}

void* t2_impl( void* _ ) {
    while( counter2 < 20000000 ) counter2++;
    sem_post( &sem );
    return NULL;
}

void* t3_impl( void* _ ) {
    sem_wait( &sem );
    sem_wait( &sem );
    printf("End: counter1 = %" PRIu64 " counter2 = %" PRIu64 "\n",
            counter1, counter2 );
    return NULL;
}

int main(void) {
    sem_init( &sem, 0, 0 );
    pthread_create( &t3, NULL, t3_impl, NULL );
    sleep( 1 );
    pthread_create( &t1, NULL, t1_impl, NULL );
    pthread_create( &t2, NULL, t2_impl, NULL );
    sem_destroy( &sem );
    pthread_exit( NULL );
    return 0;
}
```



Thesem\_initfunction initializes the semaphore. Its second argument is a flag: 0 corresponds to a process-local semaphore \(which can be used by different threads\), non-zero value sets up a semaphore visible to multiple processes.3The third argument sets up the initial semaphore value. A semaphore is deleted using thesem\_destroyfunction. In the example, two counters and three threads are created. Threadst1andt2increment the respective counters to 1000000 and 20000000 and then increment the semaphore valuesemby callingsem\_post. t3locks itself decrementing the semaphore value twice. Then, when semaphore was incremented twice by other threads,t3prints the counters intostdout.

Thepthread\_exitcall ensures that the main thread will not terminate prematurely, until all other threads finish their work.

Semaphores come up handy in such tasks as

* Forbidding more thannprocesses to simultaneously execute a code section.

* Making one thread wait for another to complete a specific action, thus imposing an order on their actions.

* Keeping no more than a fixed number of worker threads performing a certain task in parallel. More threads than needed might decrease performance.



It is not true that a semaphore with two states is fully analogous to a mutex. Unlike mutex, which can only be unlocked by the same thread that locked it, semaphores can be changed freely by any thread.

We will see another example of the semaphore usage in Listing17-18to make two threads start each loop iteration simultaneously \(and when the loop body is executed, they wait for other loops to finish an iteration\).

Manipulations with semaphores obviously act like both compiler and hardware memory barriers.

 For more information on semaphores, refer to themanpages for the following functions:

•em\_close  
•sem\_destroy

•sem\_getvalue

•sem\_init  
•sem\_open  
•sem\_post  
•sem\_unlink  
•sem\_wait



---

■Question 364 What is a named semaphore? Why should it be mandatorily unlinked even if the process is terminated?

---




17.8.5 Mutexes

While thread joining is an accessible technique, it does not provide means to control the thread execution “on the run.” Sometimes we want to ensure that the actions performed in one thread are not being performed before some other action in the other threads are performed. Otherwise, we will get a situation where the system is not always working in a stable manner: its output will become dependent on the actual order in which the instructions from the different threads will be executed. It occurs when working with the mutable data shared between threads. Such situations are calleddata races, because the threads compete for the resources, and any thread can win and get to them first.

To prevent such situations, there is a number of tools, and we will start with mutexes.

Mutex\(a shorthand for “mutual exclusion”\) is an object that can be in two states: locked and unlocked. We work with them using two queries.

* Lock. Changes the state from unlocked to locked. If the mutex is locked, then the attempting thread waits until the mutex is unlocked by other threads.

* Unlock. If the mutex is locked, it becomes unlocked.



Mutexes are often used to provide an exclusive access to a shared resource \(like a shared piece of data\). The thread that wants to work with the resource locks the mutex, which is exclusively used to control an access to a resource. After the work with the resource is finished, the thread unlocks the mutex.

Mutex locking and unlocking acts as both a compiler and full hardware memory barriers, so no reads or writes can be reordered before locking or after unlocking.

Listing17-12shows an example program which needs a mutex.

Listing 17-12.mutex\_ex\_counter\_bad.c

```
#include <pthread.h>
#include <inttypes.h>
#include <stdio.h>
#include <unistd.h>

pthread_t t1, t2;

uint64_t value = 0;

void* impl1( void* _ ) {
    for (int n = 0; n < 10000000; n++) {
        value += 1;
    }
    return NULL;
}

int main(void) {
    pthread_create(  &t1, NULL, impl1, NULL );
    pthread_create(  &t2, NULL, impl1, NULL );
    
    pthread_join( t1, NULL );
    pthread_join( t2, NULL );
    printf( "%"PRIu64"\n", value );
    return 0;
}
```

This program has two threads, implemented by the functionimpl1. Both threads are constantly incrementing the shared variablevalue 10000000times.

This program should be compiled with the optimizations disabled to prevent this incrementing loop from being transformed into a singlevalue += 10000000statement \(or we can makevalue volatile\).



```
gcc -O0 -pthread mutex_ex_counter_bad.c
```

The resulting output is, however, not 20000000, as we might have thought, and is different each time we launch the executable:

```
> ./a.out
11297520
> ./a.out
10649679
> ./a.out
13765500
```

The problem is that incrementing a variable is not an atomic operation from the C point of view. The generated assembly code conforms to this description by using multiple instructions to read a value, add one, and then put it back. It allows the scheduler to give the CPU to another thread “in the middle” of a running increment operation. The optimized code might or might not have the same behavior.

To prevent this mess we are going to use a mutex to grant a thread a privilege to be the sole one working

withvalue. This way we enforce a correct behavior. Listing17-13shows the modified program.

Listing 17-13.mutex\_ex\_counter\_good.c

```
#include <pthread.h>
#include <inttypes.h>
#include <stdio.h>
#include <unistd.h>

pthread_mutex_t m; //

pthread_t t1, t2;

uint64_t value = 0;

void* impl1( void* _ ) {
    for (int n = 0; n < 10000000; n++) {
        pthread_mutex_lock( &m );//
        
        value += 1;
        
        pthread_mutex_unlock( &m );//
    }
    return NULL;
}

int main(void) {
    pthread_mutex_init( &m, NULL );     //
    
    pthread_create(  &t1, NULL, impl1, NULL );
    pthread_create(  &t2, NULL, impl1, NULL );
    
    pthread_join( t1, NULL );
    pthread_join( t2, NULL );
    printf( "%"PRIu64"\n", value );
    
    pthread_mutex_destroy( &m ); //
    return 0;
}
```

Its output is consistent \(although takes more time to compute\):

```

> ./a.out
20000000
```

The mutexmis associatedby the programmerwith a shared variablevalue. No modifications ofvalueshould be performed outside the code section between themlock and unlock. If this constraint is satisfied, there is no wayvaluecan be changed by another thread once the lock is taken. The lock acts as a memory barrier as well. Because of that,valuewill be reread after the lock is taken and can be cached in a register safely. There is no need to make the variablevaluevolatile, since it will only suppress optimizations and the program is correct anyway.

Before a mutex can be used, it should be initialized withpthread\_mutex\_init, as seen in themainfunction. It accepts attributes, just like thepthread\_createfunction, which can be used to create a recursive mutex, create a deadlock detecting mutex, control the robustness \(what happens if the mutex owner thread dies?\), and more.

To dispose of a mutex, the call topthread\_mutex\_unlockis used.  


---

■Question 361What is a recursive mutex? how is it different from an ordinary one?

---




17.8.8 Condition Variables

Condition variables are used together with mutexes. They are like wires transmitting an impulse to wake up a sleeping thread, waiting for some condition to be satisfied.

Mutexes implement synchronization by controlling thread access to a resource; condition variables, on the other hand, allow threads to synchronize based upon additional rules. For example, in case of shared data, the actual value of data might be a part of such rule.

The core of condition variables usage consists of three new entities:

* The condition variable itself of typepthread\_cond\_t.

* A function to send a wake-up signal through a condition variablepthread\_cond\_signal.

* A function to wait until a wake-up signal comes through a condition variablepthread\_ cond\_wait.





These two functions should only be used between a lock and unlock of the same mutex.

It is an error to callpthread\_cond\_signalbeforepthread\_cond\_wait, otherwise the program might be stuck.

Let us study a minimal working example shown in Listing17-16.

Listing 17-16.condvar\_mwe.c

```
#include <pthread.h>
#include <stdio.h>
#include <stdbool.h>
#include <unistd.h>

pthread_cond_t condvar = PTHREAD_COND_INITIALIZER;
pthread_mutex_t m;

bool sent = false;
void* t1_impl( void* _ ) {
    pthread_mutex_lock( &m );
    puts( "Thread2 before wait" );
    
    while (!sent)
        pthread_cond_wait( &condvar, &m );
        
    puts( "Thread2 after wait" );
    pthread_mutex_unlock( &m );
    return  NULL;
}

void* t2_impl( void* _ ) {
    pthread_mutex_lock( &m );
    puts( "Thread1 before signal" );
    
    sent = true;
    pthread_cond_signal( &condvar );
    
    puts( "Thread1 after signal" );
    pthread_mutex_unlock( &m );
    return NULL;
}

int main( void ) {
    pthread_t t1, t2;
    
    pthread_mutex_init( &m, NULL );
    pthread_create( &t1, NULL, t1_impl, NULL );
    sleep( 2 );
    pthread_create( &t2, NULL, t2_impl, NULL );
    
    pthread_join( t1, NULL );
    pthread_join( t2, NULL );
    
    pthread_mutex_destroy( &m );
    return 0;
}
```

```
./a.out
Thread2 before wait
Thread1 before signal
Thread1 after signal
Thread2 after wait
```

Initializing a condition variable can be performed either through an assignment of a special preprocessor constantPTHREAD\_COND\_INITIALIZERor by callingpthread\_cond\_init. The latter can accept a pointer to attributes of typepthread\_condattr\_takin topthread\_createorpthread\_mutex\_init.

In this example, two threads are created:t1, performing instructions fromt1\_impl, andt2, performing ones fromt2\_impl.

The first thread locks the mutexm. It then waits for a signal that can be transmitted through the condition variablecondvar. Note thatpthread\_cond\_waitalso accepts the pointer to the currently locked mutex.

Nowt1is sleeping, waiting for the signal to come. The mutexmbecomes immediately unlocked! When the thread gets the signal, it will relock the mutex automatically and continue its execution from the next statement afterpthread\_cond\_waitcall.

The other thread is locking the same mutexmand issuing a signal throughcondvar. pthread\_cond\_ signalsends the signal throughcondvar, unblocking at least one of the threads, blocked on the condition variablecondvar.

Thepthread\_cond\_broadcastfunction would unblock all threads waiting for this condition variable, making them contend for the respective mutex as if they all issuedpthread\_mutex\_lock. It is up to the scheduler to decide in which order will they get access to the CPU.

As we see, condition variables let us block until a signal is received. An alternative would be a “busy waiting” when a variable’s value is constantly checked \(which kills performance and increases unnecessary power consumption\) as follows:

```
while (somecondition == false);
```



We can of course put the thread to sleep for a time, but this way we will still wake up either too rarely to react to the event in time or too often:

```
while (somecondition == false)
    sleep(1); /* or something else that lets us sleep for less time */

```

Condition variables let us wait just enough time and continue the thread execution in the locked state.

An important moment should be explained. Why did we introduce a shared variablesent?Why are we using it together with the condition variable? Why are we waiting inside thewhile \(!sent\)cycle?

The most important reason is that the implementation is permitted toissue spurious wake-ups to a waiting thread. It means that the thread can wake up from waiting on a signal not only after receiving it but atany time. In this case, as thesentvariable is only set before sending the signal, spurious wake-up will check its value and if it is still equal tofalsewill issue thepthread\_cond\_waitagain.


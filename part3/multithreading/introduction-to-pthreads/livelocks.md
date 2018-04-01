17.8.7 Livelocks

Livelockis a situation in which two threads are stuck but not in a waiting-for-mutex-unlock state. Their states are changing, but they are not really progressing. For example,pthreadsdoes not allow you to check whether the mutex is locked or not. It would be useless to provide information about the mutex state, because once you obtain information about it, the latter can already be changed by the other thread.

```

if ( mutex is not locked ) {
    /* We still do not know if the mutex is locked or not.
       Other thread might have locked or unlocked it
       several times already.  */
}

```

However,pthread\_mutex\_trylockis allowed, which either locks a mutex or returns an error if it has already been locked by someone. Unlikepthread\_mutex\_lock, it does not block the current thread waiting for the unlock. Usingpthread\_mutex\_trylockcan lead to livelock situations. Listing17-15shows a simple example in pseudo code.

Listing 17-15.livelock\_ex

```
mutex m1, m2;

thread1() {
    lock( m1 );
    while ( mutex_trylock m2 indicates LOCKED ) {
        unlock( m1 );
        wait for some time;
        lock( m1 );
    }
    // now we are good because both locks are taken
}

thread2() {
    lock( m2 );
    while ( mutex_trylock m1 indicates LOCKED ) {
        unlock( m2 );
        wait for some time;
        lock( m2 );
    }
    // now we are good because both locks are taken
}

```

Each thread tries to defy the principle “locks should always be performed in the same order.” Both of them want to lock two mutexesm1andm2.

The first thread performs as follows:

* Locks the mutexm1.

* Tries to lock mutexm2. On failure, unlocksm1, waits, and locksm1again.

This pause is meant to provide the other thread time to lockm1andm2and perform whatever it wants to do. However, we might be stuck in a loop when

1.Thread 1 locksm1, thread 2 locksm2.

2.Thread1seesthatm2islockedandunlocksm1foratime.

3.Thread2seesthatm1islockedandunlocksm2foratime.

4.Go back to step one.



This loop can take forever to complete or can produce significant delays; it is entirely up to the operating system scheduler. So, the problem with this code is that execution traces exist that will forever prevent threads from progress.




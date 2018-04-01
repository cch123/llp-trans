17.10 How Strong Is Intel 64?

Abstract machines with relaxed memory model can be tough to follow. Out of order writes, return values from the future, and speculative reads are confusing. Intel 64 is considered to be usually strong. In most cases, it guarantees quite a few constraints to be satisfied, including, but not limited to, the following:

* Stores are not reordered with older stores.

* Stores are not reordered with older loads.

* Loads are not reordered with other loads.

* In a multiprocessor system, stores to the same location have a total order.

  There are also exceptions, such as the following:

* Writing to memory bypassing cache with such instructions asmovntdqcan be reordered with other stores.

* String instructions likerep movscan be reordered with other stores.



A full list of guarantees can be found in volume 3, section 8.2.2 of \[15\].

However, according to \[15\], “reads may be reordered with older writes to different locations but not with older writes to the same location.” So, do not be fooled: memory reorderings do occur. A simple program shown in Listing17-18demonstrates the memory reordering done by hardware. It implements an example already shown in Listing17-4, where there are two threads and two shared variablesxandy. The first thread performsstore xandload y, the second ones performsstore yandload x. The compiler barrier ensures that these two statements are translated into assembly in the same order. As section 17.10 suggests, the stores and loads into independent locations can be reordered. So, we cannot exclude the hardware memory reordering here, asxandyare independent!



Listing 17-18.reordering\_cpu\_mwe.c

```
#include <pthread.h>
#include <semaphore.h>
#include <stdio.h>
#include <inttypes.h>
#include <stdint.h>
#include <stdlib.h>
#include <time.h>

sem_t sem_begin0, sem_begin1, sem_end;

int x, y, read0, read1;

void *thread0_impl( void *param )
{
    for (;;) {
    
        sem_wait( &sem_begin0 );
        
        x = 1;
        // This only disables compiler reorderings:
        asm volatile("" ::: "memory");

        // The following line disables also hardware reorderings:
        // asm volatile("mfence" ::: "memory");
        read0 = y;
        
        sem_post( &sem_end );
    }

    return NULL;
}

void *thread1_impl( void *param )
{
    for (;;) {
        sem_wait( &sem_begin1 );
        y = 1;
        // This only disables compiler reorderings:
        asm volatile("" ::: "memory");
        // The following line disables also hardware reorderings
        // asm volatile("mfence" ::: "memory");
        read1 = x;
        sem_post( &sem_end );
    }
    return NULL;
};

int main( void ) {

    sem_init( &sem_begin0, 0, 0);
    sem_init( &sem_begin1, 0, 0);
    sem_init( &sem_end, 0, 0);
    
    pthread_t thread0, thread1;
    pthread_create(  &thread0, NULL, thread0_impl, NULL);
    pthread_create(  &thread1, NULL, thread1_impl, NULL);
    
    for (uint64_t i = 0; i < 100000; i++)
    {
        x = 0;
        y = 0;
        sem_post( &sem_begin0 );
        sem_post( &sem_begin1 );
        sem_wait( &sem_end );
        sem_wait( &sem_end );
        if (read0 == 0 && read1 == 0 ) {
            printf( "reordering happened on iteration %" PRIu64 "\n", i );
            exit(0);
        }
    }
    puts("No reordering detected during 100000 iterations");
    return 0;
}
```

To check it we perform multiple experiments. Themainfunction acts as follows:

1. Initialize threads and two starting semaphores as well as an ending one.

2. x=0,y=0

3. Notifythethreadsthattheyshouldstartperformingatransaction.

4. Waitforboththreadstocompletethetransaction.

5. Checkwhetherthememoryreorderingtookplace.Itisseenwhenbothload xandload yreturnedzeros\(becausetheywerereorderedtobeplacedbeforestore s\).

6. If the memory reordering has been detected, we are notified about it and the process exits. Otherwise it tries again from step \(2\) up to the maximum of 100000 attempts.



Each thread waits for a signal to start frommain, performs the transaction, and notifiesmainabout it. Then it starts all over again.

After launching you will see that 100000 iterations are enough to observe a memory reordering.

```
> gcc -pthread -o ordering -O2 ordering.c
> ./ordering
reordering happened on iteration 128
> ./ordering
reordering happened on iteration 12
> ./ordering
reordering happened on iteration 171
> ./ordering
reordering happened on iteration 80
> ./ordering
reordering happened on iteration 848
> ./ordering
reordering happened on iteration 366
> ./ordering
reordering happened on iteration 273
> ./ordering
reordering happened on iteration 105
> ./ordering
reordering happened on iteration 14
> ./ordering
reordering  happened  on  iteration 5
> ./ordering
reordering happened on iteration 414
```

It might seem magical, but it is the level lower than the assembly language even that is seen here and that introduces rarely observed \(but still persistent\) bugs in the software. Such bugs in multithreaded software areveryhard to catch. Imagine a bug appearing only after four months of uninterrupted execution, which corrupts the heap and crashes the program 42 allocations after it triggers! So, writing high- performance multithreaded software in a lock-free manner requires tremendous expertise.

So, what we need to do is to addmfenceinstruction. Replacing the compiler barrier with a full memory barrierasm volatile\( "mfence":::"memory"\);solves the problem and the reorderings disappear completely. If we do it, there will be no reorderings detected no matter how many iterations we try.


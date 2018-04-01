17.8.4 Example: Distributed Factorization  


We have picked a simple CPU bound program of counting the factors of a number. First, we are going to

solve it using the most trivial brute-force method on a single core. Listing17-9shows the code.

Listing 17-9.dist\_fact\_sp.c

```
#include <pthread.h>
#include <unistd.h>
#include <inttypes.h>
#include <stdio.h>
#include <malloc.h>

uint64_t factors( uint64_t num ) {
    uint64_t result = 0;
    for (uint64_t i = 1; i <= num; i++ )
        if ( num % i == 0 ) result++;
    return result;
}

int main( void ) {
    /* volatile to prevent constant propagation */
    volatile uint64_t input = 2000000000;
    printf( "Factors of %"PRIu64": %"PRIu64"\n", input, factors(input) );
    return 0;
}

```

The code is quite simple: we naively iterate over all numbers from 1 to the input and check whether they are factors or not. Note that the input value is marked volatile to prevent the whole result from being computed during the compilation. Compile the code with the following command:

```
>
 gcc -O3 -std=c99 -o fact_sp dist_fact_sp.c

```

We will start parallelization with a dumbed-down version of multithreaded code, which will always

perform computations in two threads and will not be architecturally beautiful. Listing17-10shows it.

Listing 17-10.dist\_fact\_mp\_simple.c

```
#include <pthread.h>
#include <inttypes.h>
#include <stdio.h>

int input = 0;

int result1 = 0;
void* fact_worker1( void* arg ) {
    result1 = 0;
    for( uint64_t i = 1; i < input/2; i++ )
        if ( input % i == 0 ) result1++;
    return NULL;
}

int result2 = 0;
void* fact_worker2( void* arg ) {
    result2 = 0;
    for( uint64_t i = input/2; i <= input; i++ )
        if ( input % i == 0 ) result2++;
    return NULL;
}

uint64_t factors_mp( uint64_t num ) {
    input = num;
    pthread_t thread1, thread2;
    
    pthread_create( &thread1, NULL, fact_worker1, NULL );
    pthread_create( &thread2, NULL, fact_worker2, NULL );
    
    pthread_join( thread1, NULL );
    pthread_join( thread2, NULL );
    
    return result1 + result2;
}

int main( void ) {
    uint64_t input = 2000000000;
    printf( "Factors of %"PRIu64": %"PRIu64"\n",
            input, factors_mp(input ));
    return 0;
}
```

Upon launching it produces the same result, which is a good sign.

Factors of 2000000000: 110

What is this program doing? Well, we split the range \(0, n\] into two halves. Two worker threads are computing the number of factors in their respective halves. Then when both of them have been joined, we are guaranteed that they have already had performed all computations. The results just need to be summed up.

Then, in Listing17-11we show the multithreaded program that uses an arbitrary number of threads to compute the same result. It has a better-thought-out architecture.

Listing 17-11.dist\_fact\_mp.c

```
#include <pthread.h>
#include <unistd.h>
#include <inttypes.h>
#include <stdio.h>
#include <malloc.h>

#define THREADS 4

struct fact_task {
    uint64_t num;
    uint64_t from, to;
    uint64_t result;
};

void* fact_worker( void* arg ) {
    struct fact_task* const task =  arg;
    task-> result = 0;
    for( uint64_t i = task-> from; i < task-> to; i++ )
        if ( task->num %  i ==  0 ) task-> result ++;
    return NULL;
}

/* assuming threads_count < num */
uint64_t factors_mp( uint64_t num, size_t threads_count ) {
    struct fact_task* tasks = malloc( threads_count * sizeof( *tasks ) );
    pthread_t* threads = malloc( threads_count * sizeof( *threads ) );
    
    uint64_t start = 1;
    size_t step = num / threads_count;
    
    for( size_t i = 0; i < threads_count; i++ ) {
        tasks[i].num = num;
        tasks[i].from = start;
        tasks[i].to = start + step;
        start += step;
    }
    
    tasks[threads_count-1].to = num+1;
    for ( size_t i = 0; i < threads_count; i++ )
        pthread_create( threads + i, NULL, fact_worker, tasks + i );
        uint64_t result = 0;
    for ( size_t i = 0; i < threads_count; i++ ) {
        pthread_join( threads[i], NULL );
        result  +=  tasks[i].result;
    }
    free( tasks );
    free( threads );
    return result;
}

int main( void ) {
    uint64_t input = 2000000000;
    printf( "Factors of %"PRIu64": %"PRIu64"\n",
            input, factors_mp(input, THREADS ) );
    return 0;
}

```

Suppose we are usingtthreads. To count the number of factors ofn, we split the range from 1 tonontequal parts. We compute the number of factors in each of those intervals and then sum up the results.

We create a type to hold the information about single task calledstruct fact\_task. It includes the number itself, the range boundstoandto, and the slot for the result, which will be the number of factors ofnumbetweenfromandto.

All workers who calculate the number of factors are implemented alike, as a routinefact\_worker, which accepts a pointer to astruct fact\_task, computes the number of factors, and fills theresultfield.

The code performing thread launch and results collection is contained in thefactors\_mpfunction, which, for a given number of threads, is

* Allocating the task descriptions and the thread instances;

* Initializing the task descriptions;

* Starting all threads;

* Waiting for each thread to end its execution by usingjoinand adding up its result to the common accumulatorresult; and

* Freeing all allocated memory.



So, we put the thread creation into a black box, which allows us to benefit from the multithreading.

This code can be compiled with the following command:

```
> gcc -O3 -std=c99 -pthread -o fact_mp dist_fact_mp.c
```

The multiple threads are decreasing the overall execution time on a multicore system for this CPU bound task.

To test the execution time, we will stick with thetimeutility again \(a program, not a shell builtin command\). To ensure, that the program is being used instead of a shell builtin, we prepend it with a backslash.

```
> gcc -O3 -o sp -std=c99 dist_fact_sp.c && \time ./sp
Factors of 2000000000: 110
21.78user 0.03system 0:21.83elapsed 99%CPU (0avgtext+0avgdata 524maxresident)k
0inputs+0outputs (0major+207minor)pagefaults 0swaps
```

```
> gcc -O3 -pthread -o mp -std=c99 dist_fact_mp.c && \time ./mp
Factors of 2000000000: 110
25.48user 0.01system 0:06.58elapsed 387%CPU (0avgtext+0avgdata 656maxresident)k
0inputs+0outputs (0major+250minor)pagefaults 0swaps
```

The multithreaded program took 6.5 seconds to be executed, while the single-threaded version took almost 22 seconds. That is a big improvement.

In order to speak about performance we are going to introduce the notion of speedup.Speedupis  
 the improvement in speed of execution of a program executed on two similar architectures with different resources. By introducing more threads we make more resources available, hence the possible improvement might take place.

Obviously, for the first example we have chosen a task that is easy and more efficient to solve in parallel. The speedup will not always be that substantial, if any; however, as we see, the overall code is compact enough \(could be even less would we not take extensibility into account—for example, fix a number of threads, instead of using it as a parameter\).

---

■Question 359experiment with the number of threads and find the optimal one in your own environment.

■Question 360read about functions:pthread\_selfandpthread\_equal. Why can’t we compare threads

with a simple equality operator==?

---




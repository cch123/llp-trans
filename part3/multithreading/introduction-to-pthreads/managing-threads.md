17.8.3 Managing Threads

What we have learned is already enough to perform work in parallel. However, we have no means of synchronization yet, so once we have distributed the work for the threads, we cannot really use the computation results of one thread in other threads.

The simplest form of synchronization is thread joining. The idea is simple: by callingthread\_joinon an instance ofpthread\_twe put the current thread into the waiting state until the other thread is terminated. Listing17-8shows an example.

Listing 17-8.thread\_join\_mwe.c

```
#include <pthread.h>
#include <unistd.h>
#include <stdio.h>
void* worker( void* param ) {
    for( int i = 0; i < 3; i++ ) {
        puts( (const char*) param );
        sleep(1);
    }
    return (void*)"done";
}

int main( void ) {
    pthread_t t;
    void* result;
    pthread_create( &t, NULL, worker, (void*) "I am a worker!" );
    pthread_join( t, &result );
    puts( (const char*) result );
    return 0;
}


```

Thethread\_joinaccepts two arguments: the thread itself and the address of avoid\*variable, which will be initialized with the thread execution result.

Thread joining acts as a full barrier because we should not place before the joining any reads or writes that are planned to happen after.

By default, the threads are created joinable, but one might create a detached thread. It might bring a certain benefit: the resources of the detached thread can be released immediately upon its termination. The joinable thread, however, will be waiting to be joined before its resources can be released. To create a detached thread

•Create an attribute instancepthread\_attr\_t attr;  
•Initialize it withpthread\_attr\_init\( &attr \);  
•Callpthread\_attr\_setdetachstate\( &attr, PTHREAD\_CREATE\_DETACHED \); and•Create the thread by usingpthread\_createwith a&attras the attribute argument.

The current thread can be explicitly changed from joinable to detached by callingpthread\_detach\(\). It is impossible to do it the other way around.


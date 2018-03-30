17.8.2 Creating Threads

Creating threads is easy. Listing 17-6 shows an example.

```
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
void* threadimpl( void* arg ) {
    for(int i = 0; i < 10; i++ ) {
        puts( arg );
        sleep(1);
    }
    return NULL;
}

int main( void ) {
    pthread_t t1, t2;
    pthread_create( &t1, NULL, threadimpl, "fizz" );
    pthread_create( &t2, NULL, threadimpl, "buzzzz" );
    pthread_exit( NULL );
    puts("bye");
    return 0;
}
```

Note that the code that usespthreadlibrary must be compiled with-pthreadflag, for example,

```
> gcc -O3 -pthread main.c
```

That specifying-lpthreadwill not give us an esteemed result. Linking with the solelibpthread.ais not enough: there are several preprocessor options that are enabled by-pthread\(e.g.,\_REENTRANT\). So, whenever the-pthreadoption is available,1use it.



Initially, there is only one thread which starts executing themainfunction. Apthread\_ttype stores all information about some other thread, so that we can control it using this instance as a handle. Then, the threads are initialized withpthread\_createfunction with the following signature:



```
int pthread_create(
    pthread_t *thread,
    const pthread_attr_t *attr,
    void *(*start_routine) (void *),
    void *arg);
```

The first argument is a pointer to thepthread\_tinstance to be initialized. The second one is a collection of attributes, which we will touch later–for now, it is safe to passNULLinstead.

The thread starting function should accept a pointer and return a pointer. It accepts avoid\*pointer to its argument. Only one argument is allowed; however, you can easily create a structure or an array, which encapsulates multiple arguments, and pass a pointer to it. The return value of thestart\_routineis also a pointer and can be used to return the work result of the thread.2The last argument is the actual pointer to the argument, which will be passed to thestart\_routinefunction.

In our example, each thread is implemented the same way: it accepts a pointer \(to a string\) and then repeatedly outputs it with an interval of approximately one second. Thesleepfunction, declared inunistd.h, suspends the current thread for a given number of seconds.

After ten iterations, the thread returns. It is equivalent to calling the functionpthread\_exitwith an argument. The return value is usually the result of the computations performed by the thread; returnNULLif you do not need it. We will see later how it is possible to get this value from the parent thread.

---

■Casting tovoidConstructions such as \(void\)argchave only one purpose: suppress warnings about unused variable or argumentargc. You can sometimes find them in the source code.

---

However, the naivereturnfrom themainfunction will lead to process termination. What if other threads still exist? The main thread should wait for their termination first! This is whatpthread\_exitdoes when it is calledin the main thread: it waits for all other threads to terminate and then terminates the program. All the code that follows will not be executed, so you will not see thebyemessage instdout.

This program will output a pair ofbuzzandfizzlines in random order ten times and then exit. It is impossible to predict whether the first or the second thread will be scheduled first, so each time the order can differ. Listing17-7shows an exemplary output.



Listing 17-7.pthread\_create\_mwe\_out

```
> ./main
fizz
buzzzz
buzzzz
fizz
fizz
buzzzz
fizz
buzzzz
fizz
buzzzz
buzzzz
fizz
buzzzz
fizz
buzzzz
fizz
buzzzz
fizz
buzzzz
fizz
```

As you see, the stringbyeis not printed, because the correspondingputscall is below thepthread\_exitcall.

---

■Where are the arguments located?it is important to note that the pointer to an argument, passed to a thread, should point to the data that stays alive until the said thread is shut down. passing a pointer to stack allocated variable might be risky, since after the stack frame for the function is destroyed, accessing these deallocated variables yields undefined behavior.

unless the arguments are guaranteed to be constant \(or you intend to use them for synchronization purposes\), do not pass them to different threads.

in the example shown in listing17-6, the strings that are accepted bythreadimplare allocated in the global read-only memory \(.rodata\). thus passing a pointer to it is safe.

---

The maximum number of threads spawned depends on implementation. In Linux, for example, you can useulimit -ato get relevant information.

The threads can create other threads; there is no limitation on that.

It is indeed guaranteed by thepthreadsimplementation that a call forpthread\_createacts as a full compiler memory barrier as well as a full hardware memory barrier.

pthread\_attr\_initis used to initialize an instance of an opaque typepthread\_attr\_t\(implemented as anincomplete type\). Attributes provide additional parameters for threads such as stack size or address.

The following functions are used to set attributes:

* pthread\_attr\_setaffinity\_np–the thread will prefer to be executed on a specific CPU core.

* pthread\_attr\_setdetachstate–will we be able to callpthread\_joinon this thread, or will it be detached \(as opposed to joinable\). The purpose ofpthread\_joinwill be explained in the next section.

* pthread\_attr\_setguardsize–sets up the space before the stack limit as a region of forbidden addresses of a given size to catch stack overflows.

* pthread\_attr\_setinheritsched–are the following two parameters inherited from the parent thread \(the one where the creation happened\), or taken from the attributes themselves?

* pthread\_attr\_setschedparam–right now is all about scheduling priority, but the additional parameters can be added in the future.

* pthread\_attr\_setschedpolicy–how will the scheduling be performed. Scheduling

* policies with their respective descriptions can be seen inman sched.

* pthread\_attr\_setscope–refers to the contention scope system, which defines a set of

  threads against which this thread will compete for CPU \(or other resources\).

* pthread\_attr\_setstackaddr–where will the stack be located?

* pthread\_attr\_setstacksize–what will be the thread stack size?

* pthread\_attr\_setstack–sets both stack address and stack size.



All of them have their “get” counterparts \(e.g.,pthread\_attr\_getscope\).

---

■Question 357readmanpages for the functions listed earlier.

■Question 358What willsysconf\(\_SC\_NPROCESSORS\_ONLN\)return?

---




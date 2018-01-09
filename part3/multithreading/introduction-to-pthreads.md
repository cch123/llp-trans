17.8 Introduction to pthreads

POSIX threads \(pthreads\) is a standard describing a certain model of program execution. It provides means to execute code in parallel and to control the execution. It is implemented as a librarypthreads, which we are going to use throughout this chapter.

The library contains C types, constants, and procedures \(which are prefixed withpthread\_\). Their declarations are available in thepthread.hheader. The functions provided by it fall into one of the following groups:

* Basic thread management \(creating, destroying\).

* Mutex management.

* Condition variables.

* Synchronization using locks and barriers.

In this section we are going to study several examples to become familiar withpthreads. To perform multithreaded computations you have the following two options:

* Spawn multiple threads in the same process.  The threads share the same address space, so the data exchange is relatively easy and fast. When the process terminates, so do all of its threads.

* Spawn multiple processes; each of them has its own default thread. These threads communicate via mechanisms provided by the operating system \(such as pipes\).

  This is not that fast; also spawning a process is slower than spawning just a thread, because it creates more operating system structures \(and a separate address space\). The communication between processes often implies one or more \(sometimes implicit\) copy operations.

  However, separating program logic into separate processes can have a positive impact on security and robustness, because each thread only sees the exposed part of the processes others than its own.

Pthreads allows you to spawn multiple threads in a single process, and that is what you usually want to do.


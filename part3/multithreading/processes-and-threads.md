17.1 Processes and Threads

It is important to understand the difference between two key concepts involved in most talks about multithreading: threads and processes.

Aprocessis a resource container that collects all kinds of runtime information and resources a program needs to be executed. A process contains the following:

* An address space, partially filled with executable code, data, shared libraries, other mapped files, etc. Parts of it can be shared with other processes.

* All other kinds of associated state such as open file descriptors, registers, etc.

* Information such as process ID, process group ID, user ID, group ID . . .

* Other resources used for interprocess communication: pipes, semaphores, message queues . . .

threadis a stream of instructions that can be scheduled for execution by the operating system.

The operating system does not schedule processes but threads. Each thread lives as a part of a process and has a piece of process state, which is its own.

* Registers.
* Stack \(technically, it is defined by the stack pointer register; however, as all processor’s threads share the same address space, one of them can access the stacks of other threads, although this is rarely a good idea\).
* Properties of importance to the scheduler such as priority.

* Pending and blocked signals.

* Signal mask.

When the process is closed, all associated resources are freed, including all its threads, open file descriptors, etc.


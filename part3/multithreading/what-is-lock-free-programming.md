17.11 What Is Lock-Free Programming?

We have seen how we can ensure the consistency of the operations when working in a multithreaded environment. Every time we need to perform a complex operation on shared data or resources without other threads intervening we lock a mutex that we have associated to this resource or memory chunk.

We say that the code islock-freeif the following two constraints are satisfied:

* No mutexes are used.

* The system cannot be locked up indefinitely. That includes livelocks.

In other words, it is a family of techniques that ensure safe manipulation with the shared data without using mutexes.



We almost always expect only a part of the program code to satisfy the lock-free property. For example, a data structure, such as a queue, may be considered lock-free if the functions that are used to manipulate it are lock-free. So, it does not prevent us from locking up completely, but as far as we are calling functions such asenqueueordequeue, progress will be made.

From the programmer’s perspective, lock-free programming is different from traditional mutex usage because it introduces two challenges that are usually covered by mutexes.

1.Reorderings. While mutex manipulations imply compiler and hardware memory barriers, without them you have to be specific about where to place memory barriers. You will not want to place them after each statement because it kills performance.

2.Non-atomic operations. The operations between mutex lock and unlock are safe and atomic in a sense. No other thread can modify the data associated with the mutex \(unless there are unsafe data manipulations outside the lock-unlock section\). Without that mechanism we are stuck with very few atomic operations, which we will study later in this chapter.

On most modern processors reads and writes of naturally aligned native types are atomic. Natural alignment means aligning the variable to a boundary that corresponds to its size.

On Intel 64 there is no guarantee that reads and writes larger than 8 bytes are atomic. Other memory interactions are usually non-atomic. It includes, but is not limited to,

* 16-byte reads and writes performed by SSE \(Streaming SIMD Extensions\) instructions.

* String operations \(movsbinstruction and the like\).

* Many operations are atomic on a single-processor system but not in a multiprocessor system \(e.g.,incinstruction\).



Making them atomic requires a speciallockprefix to be used, which prevents other processors from doing their own read-modify-write sequence between the stages of the said instructions. Aninc&lt;addr&gt; instruction, for instance, has to read bytes from memory and write back their incremented value. Withoutlockprefix, they can intervene in between, which can lead to a loss of value.

Here are some examples of non-atomic operations:

```
char buf[1024];
uint64_t* data = (uint64_t*)(buf + 1);

/* not atomic: unnatural alignment */
*data = 0;

/* not atomic: increment can need a read and a write */
++global_aligned_var;

/* atomic write */

global_aligned_var = 0;

void f(void) {
/* atomic read */
int64_t local_variable = global_aligned_var;
}

```

These cases are architecture-specific. We also want to perform more complex operations atomically \(e.g., incrementing the counter\). To perform them safely without using mutexes the engineers invented interesting basic operations, such as compare-and-swap \(CAS\). Once this operation is implemented as a machine instruction on a specific architecture, it can be used in combination with more trivial non-atomic reads and writes to implement many lock-free algorithms and data structures.

CAS instruction acts as an atomic sequence of operations, described by the following equivalent C function:



```
bool cas(int* p , int old, int new) {
    if (*p != old) return false;
    *p = new;
    return true;
}
```

A shared counter, which you are reading and writing back a modified value, is a typical case when we need a CAS instruction to perform an atomical increment or decrement. Listing17-19shows a function to perform it.



Listing 17-19.cas\_counter.c

```
int add(int* p, int add ) {
    bool done = false;
    int value;
    while (!done) {
        value = *p;
            done = cas(p, value, value + add );
    }
    return value + add;
}
```

This example shows a typical pattern, seen in many CAS-based algorithms. They read a certain memory location, compute its modified value, and repeatedly try to swap the new value back if the current memory value is equal to the old one. It fails in case this memory location was modified by another thread; then the whole read-modify-write cycle is repeated.

Intel 64 implements CAS instructionscmpxchg,cmpxchg8b, andcmpxchg16b. In case of multiple processors, they also require alockprefix to be used.

The instructioncmpxchgis of a particular interest. It accepts two operands: register or memory and a register. It comparesrax4with the first operand. If they are equal,zfflag is set, the second operand’s value is loaded into the first. Otherwise, the actual value of the first operand is loaded intoraxandzfis cleared.

These instructions can be used as a part of implementation of mutexes and semaphores.

As we will see in section 17.12.2, there is now a standard-compliant way of using compare-and-set operations \(as well as manipulating with atomic variables\). We recommend sticking to it to prevent non- portable code and use atomics whenever you can. When you need complex operations to be performed atomically, use mutexes or stick with the lock-free data structure implementations done by experts: writing lock-free data structures has proven to be a challenge.

---

■Question 365 What is the aBa problem?  
■Question 366 read the description ofcmpxchgin intel docs \[15\].

---




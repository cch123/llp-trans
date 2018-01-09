17.12.2 Atomics

The important C11 feature that can be used to write fast multithreaded programs is atomics \(see section 7.17 of \[7\]\). These are special variable types, which can be modified atomically. To use them, include the headerstdatomic.h.

Apparently, an architecture support is needed to implement them efficiently. In the worst-case scenario, when the architecture does not support any such operation, every such variable will be paired with a mutex, which will be locked to perform any modification of the variable or even to read it.

Atomics allow us to perform thread-safe operations on common data in some cases. It is often possible to do without heavy machinery involving mutexes. However, writing data structures such as queue in a lock- free way is no easy task. For that we highly advise using such existing implementations as “black boxes.”

C11 defines a new\_Atomic\(\)type specifier. You can declare an atomic integer as follows:

\_Atomic\(int\) counter;

\_Atomictransforms the name of a type into the name of an atomic type. Alternatively, you can use the atomic types directly as follows:

atomic\_int counter;

A full correspondence between\_Atomic\(T\)andatomic\_Tdirect type forms can be found in section 7.17.6 of \[7\].

Atomiclocalvariables shouldnotbe initialized directly; instead, the macroATOMIC\_VAR\_INITshould be used. It is understandable, because on some architectures with fewer hardware capabilities each such variable should be associated with a mutex, which has to be created and initialized as well. Global atomic variables are guaranteed to be in a correct initial state.ATOMIC\_VAR\_INITshould be used during the variable declaration coupled with initialization; however, if you want to initialize the variable later, useatomic\_initmacro.

```
void f(void) {
    /* Initialization during declaration */
    atomic_int x = ATOMIC_VAR_INIT( 42 );
    atomic_int y;
    
    /* initialization later */
    atomic_init(&y, 42 );
}
```



It is your responsibility to guarantee that the atomic variable initialization ends before anything else is done with it. In other words, concurrent access to the variable being initialized is a data race.

Atomic variables should only be manipulated through an interface, defined in the language standard. It consists of several operations, such as load, store, exchange, etc. Each of them exists in two versions.

An explicit version, which accepts an extra argument, describing the memory ordering. Its name ends with\_explicit. For example, the load operation is

* ```
           T atomic_load_explicit( _Atomic(T) *object, memory_order order );

  ```
* An implicit version, which implies the strongest memory ordering \(sequentially consistent\). There is no\_explicitsuffix. For example,

```
         T atomic_load( _Atomic(T) *object );
```






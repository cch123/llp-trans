17.6 What Is Volatile and What Is Not

The C memory model, which we are using, is quite weak. Consider the following code:

```
int x,y; x = 1;
y = 2;
x = 3;
```

As we have already seen, the instructions can be reordered by the compiler. Even more, the compiler can deduce that the first assignment is the dead code, because it is followed in another assignment to the same variablex. As it is useless, the compiler can even remove this statement.

The volatile keyword addresses this issue. It forces the compiler to never optimize the writes and reads to the said variable and also suppresses any possible instruction reorderings. However, it only enforces these restrictions to one single variable and gives no guarantee about the order in which writes to different volatile variables are emitted. For example, in the previous code, even changing bothxandytype tovolatile intwill impose an order on assignments of each of them but will still allow us to interleave the writes freely as follows:

```
volatile int x, y;
x = 1;
x = 3;
y = 2;
```

Or like this:

```
volatile int x, y;
y = 2;
x = 1;
x = 3;
```

Obviously, these guarantees are not sufficient for multithreaded applications. You cannot usevolatilevariables to organize an access to the shared data, because these accesses can be moved around freely enough.

To safely access shared data, we need two guarantees.

The read or write actually takes place. The compiler could have just cached the value in the register and never write it back to memory.

* This is the guaranteevolatileprovides. It is enough to perform memory-mapped I/O \(input/output\), but not for multithreaded applications.

* No memory reordering should take place. Let us imagine we usevolatilevariable as a flag, which indicates that some data is ready to be read. The code prepares data and then sets the flag; however, reordering can place this assignment before the data is prepared.

  Both hardware and compiler reorderings matter here. This guarantee is not provided byvolatilevariables.

In this chapter, we study two mechanisms that provide both of the following guarantees:

* Memory barriers.

* Atomic variables, introduced in C11.

Volatile variables are used extremely rarely. They suppress optimization, which is usually not something we want to do.




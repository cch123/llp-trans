17.12.4 Operations

The following operations can be performed on atomic variables \(Tdenotes the non-atomic type,Urefers to the type of the other argument for arithmetic operations; for all types except pointers, it is the same asT, for pointers it isptrdiff\_t\).

```
void atomic_store(volatile _Atomic(T)* object, T  value);
T atomic_load(volatile _Atomic(T)* object);
T atomic_exchange(volatile _Atomic(T)* object, desired);
T atomic_fetch_add(volatile _Atomic(T)* object, U operand);
T atomic_fetch_sub(volatile _Atomic(T)* object, U operand);
T atomic_fetch_or (volatile _Atomic(T)* object, U operand);
T atomic_fetch_xor(volatile _Atomic(T)* object, U operand);
T atomic_fetch_and(volatile _Atomic(T)* object, U operand);
```

```
bool atomic_compare_exchange_strong(
    volatile _Atomic(T)* object, T * expected, T desired);
bool atomic_compare_exchange_weak(
    volatile _Atomic(T)* object, T * expected, T desired);
```

All these operations can be used with an\_explicitsuffix to provide a memory ordering as an additional argument.

Load and store functions do not need a further explanation; we will discuss the other ones briefly.

atomic\_exchangeis a combination of load and store: it replaces the value of an atomic variable withdesiredand returns its old value.

fetch\_opfamily of operations is used to atomically change the atomic variable value. Imagine you need to increment an atomic counter. Withoutfetch\_addit is impossible to do since in order to increment it you need to add one to its old value, which you have to read first. This operation is performed in three steps: reading, addition, writing. Other threads may interfere between these stages, which destroys atomicity.

atomic\_compare\_exchange\_strongis preferred to its weak counterpart, since the weak version can fail spuriously. The latter has a better performance on some platforms.

Theatomic\_compare\_exchange\_strongfunction is roughly equivalent to the following pseudo code:

```
if ( *object == *expected )
    *object = desired;
else
    *expected = *object;
```

As we see, this is a typical CAS instruction that was discussed in section 17.11.atomic\_is\_lock\_freemacro is used to check whether a specific atomic variable uses locks or not. Remember that without providing explicit memory ordering,allthese operations are assumed to be

sequentially consistent, which in Intel 64 meansmfenceinstructions all over the code. This can be a huge performance killer.

The Boolean shared flag has a special type namedatomic\_flag. It has two states: set and clear. It is guaranteed that operations on it are atomic without using locks.

The flag should be initialized with theATOMIC\_FLAG\_INITmacro as follows:

atomic\_flag is\_working = ATOMIC\_FLAG\_INIT;

The relevant functions areatomic\_flag\_test\_and\_setandatomic\_flag\_clear, both of which have\_explicitcounterparts, accepting memory ordering descriptions.

---

■Question 367 read man pages for atomic\_flag\_test\_and\_set and atomic\_flag\_clear.

---




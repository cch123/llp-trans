14.2.1 Lazy Memory Allocation

Many operating systems map pages lazily, at the time of the first usage rather than right aftermmapcall \(or its equivalent\).

If the programmer wants no delays on the first-page usages, he might choose to address each page individually so that the operating system really creates it, as shown in Listing14-9.

Listing 14-9.lma\_bad.c

```
char* ptr;
for( ptr = start; ptr < start + size; ptr += pagesize )
*ptr;
```

However, this code has no observable effect from the point of view of the compiler, so it might be optimized away completely. However, when the pointer is markedvolatile, this will not be the case. Listing14-10shows an example.

Listing 14-10.lma\_good.c

```
volatile char* ptr;
for( ptr = start; ptr < start + size; ptr += pagesize )
*ptr;
```

---

■Volatile pointers in the language standard if the volatile pointer is pointing at the non-volatile memory, according to the standard there are no guarantees! they exist only when both the pointer and the memory are volatile. so, according to the standard, the example above is incorrect. however, as programmers are using the volatile pointers with exactly this reasoning, the most used compilers \(MsVC, GCC, clang\) do not optimize away the dereferencing of volatile pointers. there is no a standard-conforming way of doing this.

---




14.5 restrict

restrictis a keyword akin tovolatileandconstwhich first appeared in the C99 standard. It is used to mark pointers and is thus placed to the right of the asterisk, as follows:

```
int x;
int* restrict p_x = &x;
```

If we create a restricted pointer to an object, we make a promise that all accesses to this object will pass through the value of this pointer. A compiler can either ignore this or make use of it for certain optimizations, which is often possible.

In other words, any write by another pointer will not affect the value stored by a restricted pointer. Breaking this promise leads to subtle bugs and is a clear case of undefined behavior.  
 Withoutrestrict, every pointer is a source of possible memory aliasing, when you can access the same

memory cells by using different names for them. Consider a very simple example, shown in Listing14-18. Is the body offequal to\*x += 2 \* \(\*add\);?

Listing 14-18. restrict\_motiv.c

```
void f(int* x, int* add) {
    *x += *add;*x += *add;
}
```

The answer is, surprisingly, no, they are not equal. What ifaddandxare pointing to the same address? In this case, changing\*xchanges\*addas well. So, in casex == add, the function will add\*xto\*xmaking it two times the initial value, and then repeat it making it four times the initial value. However, whenx != add, even if\*x == \*addthe final\*xwill be three times the initial value.

The compiler is well aware of it, and even with optimizations turned on it will not optimize away two reads, as shown in Listing14-19.

Listing 14-19.restrict\_motiv\_dump.asm

```
0000000000000000 <f>:
0:   8b 06               mov   eax,DWORD PTR [rsi]
2:   03 07               add   eax,DWORD PTR [rdi]
4:   89 07               mov   DWORD PTR [rdi],eax
6:   03 06               add   eax,DWORD PTR [rsi]
8:   89 07               mov   DWORD PTR [rdi],eax
a:   c3                  ret
```

However, addrestrict, as shown in Listing14-20, and the disassembly will demonstrate an improvement, as shown in Listing14-21. The second argument is read exactly once, multiplied by 2, and added to the dereferenced first argument.

Listing 14-20.restrict\_motiv1.c

```
void f(int* restrict x, int* restrict add) {
    *x += *add;
    *x += *add;
}
```

Listing 14-21.restrict\_motiv\_dump1.asm

```
0000000000000000 <f>:
   0:   8b 06              mov   eax,DWORD PTR [rsi]
   2:   01 c0              add   eax,eax
   4:   01 07              add   DWORD PTR [rdi],eax
   6:   c3                 ret
```

Only userestrictif you are sure what you are doing. Writing a slightly ineffective program is much better than writing an incorrect one.

It is important to userestrictalso to document code. For example, the signature formemcpy, a function that copiesnbytes from some starting addresss2to a block starting withs1, has changed in C99:

```
void*
memcpy(void*       restrict s1,
       const void* restrict s2,

       size_t              n );
```

This reflects the fact that these two areas should not overlap; otherwise the correctness is not guaranteed.

Restricted pointers can be copied from one to another to create a hierarchy of pointers. However, the standard limits this by cases when the copy is not residing in the same block with the original pointer. Listing14-22shows an example.

Listing 14-22.restrict\_hierarchy.c

```
struct s {
    int* x;
} inst;

void f(void) {
    struct s* restrict p_s = &inst;
    int* restrict p_x = p_s->x; /* Bad */
    {
        int* restrict p_x2 = p_s->x; /* Fine, other block scope */
    }
}
```




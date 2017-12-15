16.1.6 Constant Propagation

Related GCC options:-fipa-cp, -fgcse, -fipa-cp-clone, etc.  
 If compiler can prove that a variable has a specific value in a certain place of the program, it can omit

reading its value and put it there directly.  
 Sometimes it even generates specialized function versions, partially applied to some arguments, if it

knows an exact argument value \(option-fipa-cp-clone\). For example, Listing16-10shows the typical case when a specialized function version will be created for sum, which has only one argument instead of two, and the other argument’s value is fixed and equal to 42.

Listing 16-10.constant\_propagation.c \_\_attribute\_\_ \(\(noinline\)\)

```
static int sum(int x, int y) { return x + y; }

int main( int argc, char** argv ) {
    return sum( 42, argc );
}
```

Listing16-11shows the translated assembly code.

Listing 16-11.constant\_propagation.asm

```
00000000004004c0   <sum.constprop.0>:
4004c0:       8d 47 2a                     lea       eax,[rdi+0x2a]
4004c3:       c3                           ret

```

It gets better when the compiler computes complex expressions for you \(including function calls\). Listing16-2shows an example.

Listing 16-12.cp\_fact.c

\#include &lt;stdio.h&gt;

```
int fact( int n ) {
    if (n == 0) return 1;
    else return n * fact( n-1 );
}

int main(void) {
    printf("%d\n", fact( 4 ) );
    return 0;
}

```

Obviously, the factorial function will always compute the same result, because this value does not depend on user input. GCC is smart enough to precompute this value erasing the call and substituting thefact\(4\)value directly with 24, as shown in Listing16-13. The instructionmov edx,0x18places 2410= 1816directly intordx.

Listing 16-13.cp\_fact.asm

```
0000000000400450 <main>:
400450:  48 83 ec 08         sub      rsp,0x8
400454:  ba 18 00 00 00      mov      edx,0x18
400459:  be 44 07 40 00      mov      esi,0x400744
40045e:  bf 01 00 00 00      mov      edi,0x1
400463:  31 c0               xor      eax,eax
400465:  e8 c6 ff ff ff      call     400430 <__printf_chk@plt>
40046a:  31 c0               xor      eax,eax
40046c:  48 83 c4 08         add      rsp,0x8
400470:  c3                  ret

```



How do we use it?Named constants are not harmful, nor are constant variables. A compiler can and will precompute as much as it is able to, including functions without side effects launched on known arguments.

Multiple function copies for each distinct argument value can be bad for locality and will make the executable size grow. Take that into account if you face performance issues.


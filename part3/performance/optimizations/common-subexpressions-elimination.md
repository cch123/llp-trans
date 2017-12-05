16.1.5 Common Subexpressions Elimination

Related GCC options:-fgcseand others containingcsesubstring.  
 Computing two expressions with a common part does not result in computing this part twice. It means

that it makes no sense performance-wise to compute this part ahead, store its result in a variable, and use it in two expressions.

In an example shown in Listing16-8a subexpressionx2+ 2xis computed once, while the naive approach suggests otherwise.

Listing 16-8.common\_subexpression.c

```
#include <stdio.h>

__attribute__ ((noinline))
    void test(int x) {
        printf("%d %d",
                x*x + 2*x + 1,
                x*x + 2*x - 1 );
    }
    
int main(int argc, char** argv) {
    test( argc );
    return 0;
}
```

As a proof, Listing16-9shows the compiled code, which does not computex2+ 2xtwice.

Listing 16-9.common\_subexpression.asm

```
0000000000400516 <test>:
; rsi = x + 2
400516:       8d 77 02                           lea       esi,[rdi+0x2]
400519:       31 c0                              xor       eax,eax
40051b:       0f af f7                           imul      esi,edi
; rsi = x*(x+2)
40051e:       bf b4 05 40 00                     mov       edi,0x4005b4
; rdx = rsi-1 = x*(x+2) - 1
400523:       8d 56 ff                           lea       edx,[rsi-0x1]
; rsi = rsi + 1 = x*(x+2) - 1
400526:       ff c6                              inc       esi
400528:       e9 b3 fe ff ff                     jmp       4003e0 <printf@plt>
```

How do we use it?Do not be afraid to write beautiful formulae with same common subexpressions: they will be computed efficiently. Favor code readability.


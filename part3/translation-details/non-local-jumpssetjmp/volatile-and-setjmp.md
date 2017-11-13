14.3.1 Volatile and setjmp

The compiler thinks thatsetjmpis just a function. However, this is not really so, because this is the point from which the program might start to execute again. In normal conditions, some local variables might have been cached in registers \(or never allocated\) before the call tosetjmp. When we return to this point due to alongjmpcall, they will not be restored.

Turning off optimizations changes this behavior. So optimizations turned off hide bugs related tosetjmpusage.

To write correctly, remember that onlyvolatilelocal variables are holding defined values afterlongjmp. They arenotrestored to their ancient values, becausejmp\_bufdoes not save stack variables but keeps the values from beforelongjmp.

Listing14-15shows an example.

Listing 14-15.setjmp\_volatile.c

```
#include <stdio.h>
#include <setjmp.h>

jmp_buf buf;

int main( int argc, char** argv ) {
    int var = 0;
    volatile int b = 0;
    setjmp( buf );
    if (b < 3) {
        b++;
        var ++;
        printf( "\n\n%d\n", var );
        longjmp( buf, 1 );
    }
    return 0; 
}

```

We are going to compile it without optimizations\(gcc -O0,Listing14-16\)and with optimizations \(gcc -O2, Listing14-17\).

Without optimizations,

Listing 14-16.volatile\_setjmp\_o0.asm

    main:
    push     rbp
    mov      rbp,rsp
    sub      rsp,0x20

    ; `argc` and `argv` are saved in stack to make `rdi` and `rsi` available
    mov    DWORD PTR [rbp-0x14],edi
    mov    QWORD PTR [rbp-0x20],rsi

    ; var = 0
    mov    DWORD PTR [rbp-0x4],0x0

    ;b= 0
    mov DWORD PTR [rbp-0x8],0x0

    ; 0x600a40 is the address of `buf` (a global variable of type `jmp_buf`)
    mov    edi,0x600a40
    call   400470 <_setjmp@plt>

    ; if (b < 3), the good branch is executed
    ; This is encoded by skipping several instructions to the `.endlabel` if b > 2
    mov    eax,DWORD PTR [rbp-0x8]
    cmp    eax,0x2
    jg     .endlabel

    ; A fair increment
    ; b++
    mov    eax,DWORD PTR [rbp-0x8]
    add    eax,0x1
    mov    DWORD PTR [rbp-0x8],eax

    ; var++
    add    DWORD PTR [rbp-0x4],0x1

    ; `printf` call
    mov    eax,DWORD PTR [rbp-0x4]
    mov    esi,eax
    mov    edi,0x400684
    ; There are no floating point arguments, thus rax = 0
    mov    eax,0x0
    call   400450 <printf@plt>

    ; calling `longjmp`
    mov    esi,0x1
    mov    edi,0x600a40
    call   400490 <longjmp@plt>

    .endlabel:
    Mov    eax,0x0
    Leave
    ret

The program output will be

1

2

3

With optimizations,

Listing 14-17.volatile\_setjmp\_o2.asm

    main:

    ; allocating memory in stack
    sub    rsp,0x18

    ; a `setjmp` argument, the address of `buf`

    mov    edi,0x600a40

    ;b= 0
    mov DWORD PTR [rsp+0xc],0x0
    ; instructions are placed in the order different
    ; from C statements to make better use of pipeline and other inner ; CPU mechanisms.
    call 400470 <_setjmp@plt>

    ; `b` is read and checked in a fair way
    mov    eax,DWORD PTR [rsp+0xc]
    cmp    eax,0x2
    jle    .branch

    ; return 0
    xor    eax,eax
    add    rsp,0x18
    ret

    .branch:

    mov    eax,DWORD PTR [rsp+0xc]

    ; the second argument of `printf` is var + 1
    ; It was not even read from memory nor allocated.
    ; The computations were performed in compile time
    mov    esi,0x1

    ; The first argument of `printf`
    mov    edi,0x400674

    ;b=b+ 1
    add eax,0x1
    mov DWORD PTR [rsp+0xc],eax

    xor eax,eax
    call   400450 <printf@plt>

    ; longjmp( buf, 1 )
    mov    esi,0x1
    mov    edi,0x600a40
    call   400490 <longjmp@plt>


The program output will be

1

1

1

The volatile variableb, as you see, behaved as intended \(otherwise, the cycle would have never ended\). The variablevarwas always equal to 1, despite being “incremented” according to the program text.

---

■Question 265 how do you implement “try–catch”-alike constructions using setjmp and longjmp?

---




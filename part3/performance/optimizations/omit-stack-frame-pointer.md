16.1.3 Omit Stack Frame Pointer

Related GCC options:-fomit-frame-pointer  
Sometimes we do not really need to store oldrbpvalue and initialize it with the new base value. It

happens when

* There are no local variables.

* Local variables fit into the red zone AND the function calls no other function.



However, there is a downside: it means that less information about the program state is kept at runtime. We will have trouble unwinding call stack and getting local variable values because we lack information about where the frame starts.

It is most troublesome in situations in which a program crashed and a dump of its state should be analyzed. Such dumps are often heavily optimized and lack debug information.

Performance-wise the effects of this optimizations are often negligible \[26\].

The code shown in Listing16-1demonstrates how to unwind the stack and display frame pointer addresses for all functions launched whenunwindgets called. Compile it with-O0to prevent optimizations.

_**Listing 16-1**.stack\_unwind.c_

```
void unwind();
void f( int count ) {
    if ( count ) f( count-1 ); else unwind();
}
int main(void) {
    f( 10 ); return 0;
}
```

Listing16-2shows an example.

Listing 16-2.stack\_unwind.asm

```
extern printf
global unwind

section .rodata
format : db "%x ", 10, 0

section .code
unwind:
push rbx

; while (rbx != 0) {
    ; print rbx; rbx = [rbx];
    ;}
    mov rbx, rbp
    .loop:
    test rbx, rbx
    jz .end
    mov rdi, format
    mov rsi, rbx
    call printf
    mov rbx, [rbx]
    jmp .loop
    
    .end:
    pop rbx
    ret
```

How do we use it?Try it as a last resort to improve performance on code involving a huge amount of non-inlineable function calls.


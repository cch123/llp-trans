14.1.3 Example: Simple Function and Its Stack

Let’s take a look at a simple function that calculates a maximum of two values. We are going to compile it without optimizations and see the assembly listing.

Listing14-4shows an example.

Listing 14-4.maximum.c

```
int maximum( int a, int b ) {
    char buffer[4096];
    if (a < b) return b;
    return a;
}

int main(void) {
    int x = maximum( 42, 999 );
    return 0;
}

```

Listing14-5shows the disassembly produced byobjdump.

Listing 14-5.maximum.asm





After a bit of cleaning, we get a pure and more readable assembly code, which is shown in Listing14-6.

Listing 14-6.maximum\_refined.asm



---

■Register assignment Refer to section 3.4.2 for the explanation about why changingesimeans a change in the whole rsi.

---

We are going to trace the function call and its prologue \(check Listing14-6\) and show the stack contents immediately after its execution.



call maximum

push rbp

mov rbp, rsp

sub rsp, 3984




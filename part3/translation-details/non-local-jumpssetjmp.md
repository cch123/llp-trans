14.3 Non-Local jumps–setjmp

The standard C library contains machinery to perform a very tricky kind of hack. It allows storing a computation context and restoring it. The context describes the program execution statewith the exception ofthe following:

* Everything related to the external world \(e.g., opened descriptors\).

* Floating point computations context.

* Stack variables.

It allows saving context and jumping back to it in case we feel like we have to return. We are not limited by the same function scope.

Include thesetjmp.hto gain access to the following machinery:

* jmp\_bufis a type of a variable which can store the context.
* int setjmp\(jmp\_buf env\) is a function that accepts ajmp\_bufinstance and stores the current context in it. By default it returns 0.

* void longjmp\(jmp\_buf env, int val\) is used to return to a saved context, stored in a certain variable of typejmp\_buf.

When returning from thelongjmp,setjmpreturns not necessarily 0 but the valuevalfed tolongjmp. Listing14-13shows an example. The firstsetjmpwill return 0 by default and so will be thevalvalue. However, thelongjmpaccepts 1 as its argument, and the program execution will continue from thesetjmpcall \(because they are linked through the usage of thejb\). This timesetjmpwill return 1 and this is the value that will be assigned toval.



Listing 14-13.longjmp.c

```
#include <stdio.h>
#include <setjmp.h>

int main(void) {
    jmp_buf jb;
    int val;
    val = setjmp( jb );
    puts("Hello!");
    if (val == 0) longjmp( jb, 1 );
    else puts("End");
    return 0;
}
```

Local variables that are not markedvolatilewill all hold undefined values afterlongjmp. This is the source of bugs as well as memory freeing related issues: it is hard to analyze the control flow in presence oflongjmpand ensure that all dynamically allocated memory is freed.

In general, it is allowed to callsetjmpas a part of a complex expression, but only in rare cases. In most cases, this is an undefined behavior. So, better not to do it.

It is important to remember that all this machinery is based on stack frames usage. It means that you cannot performlongjmpin a function with a deinitialized stack frame. For example, the code, shown in Listing14-14, yields an undefined behavior for this very reason.

Listing 14-14.longjmp\_ub.c

```
jmp_buf jb;
void f(void) {
    setjmp( jb );
}

void g(void) {
    f();
    longjmp(jb);
}
```

The functionfhas terminated already, but we are performing longjmp into it. The program behavior is undefined because we are trying to restore a context inside a destroyed stack frame.

In other words, you can only jump into the same function or into a function that is launched.


14.1.5 Variable Number of Arguments

The calling convention that we are using supports the variable arguments count. It means that the function can accept an arbitrary number of arguments. It is possible because arguments passing \(and cleaning the stack after the function termination\) is the responsibility of the calling function.

The declaration of such functions contains a so-called ellipsis—three dots instead of the last argument. The typical function with variable number of arguments is our old friendprintf.

```
void printf( char const* format, ... );
```

How does printf know the exact number of arguments? It knows for sure that at least one argument is passed \(char const\* format\). By analyzing this string and counting the specifiers it will compute the total number of arguments as well as their types \(in which registers they should be\).

---

■Note in case of variable number of arguments,alshould contain the number ofxmmregisters used by arguments.

---

As you see, there is absolutely no way to know how many arguments have been exactly passed. The function deduces it from the arguments that are certainly present \(formatin this case\). If there are more format specifiers than arguments,printfwill not know about it and will try to get the contents of the respective registers and memory naively.

Apparently, this functionality cannot be encoded in C by a programmer directly, because the registers cannot be accessed directly. However, there is a portable mechanism of declaring functions with variable argument count that is a part of the standard library. Each platform has its own implementation of this mechanism. It can be used afterstdarg.hfile is included and consists of the following:

* va\_list–a structure that stores information about arguments.

* va\_start–amacrothat initializesva\_list.

* va\_end–amacrothat deinitializesva\_list.

* va\_arg–amacrothat takes a next argument from the argument list when given an instance ofva\_listand an argument type.

Listing14-7shows an example. The functionprinteraccepts a number of arguments and an arbitrary number of them.

Listing 14-7.vararg.c

```
#include <stdarg.h>
#include <stdio.h>

void printer( unsigned long argcount, ... ) {
    va_list args;
    unsigned long i;
    va_start( args, argcount );
    for (i = 0; i < argcount; i++ )
        printf(" %d\n", va_arg(args, int )  );
    va_end( args );
}

int main () {
    printer(10, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0 );
    return 0;
}
```

First,va\_listis initialized with the name of the last argument before dots byva\_start. Then, each call tova\_arggets the next argument. The second parameter is the name of the fresh argument’s type. In the end,va\_listis deinitialized usingva\_end.

Since a type name becomes an argument andva\_listis used by name, but is mutated, this example can look confusing.

---

■Question 264 Can you imagine a situation in which a function, not a macro, accepts a variable by name \(syntactically\) and changes it? What should be the type of such variable?

---




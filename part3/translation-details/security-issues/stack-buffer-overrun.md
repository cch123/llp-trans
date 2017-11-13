14.7.1 Stack Buffer Overrun  


Suppose that the program uses a functionfwith a local buffer, as shown in Listing14-25.

Listing 14-25.buffer\_overrun.c

```
#include <stdio.h>

void f( void ) {
    char buffer[16];
    gets( buffer );
}

int main( int argc, char** argv ) {
    f();
    return 0;
}
```

After being initialized, the layout of the stack frame will look as follows:

14-g?

The gets function reads a line fromstdinand places it in the buffer, whose address is accepted as an argument. Unfortunately, it does not control the buffer size at all and thus can surpass it.

If the line is too long, it will overwrite the buffer, then the savedrbpvalue, and then the return address. When theretinstruction is executed, the program will most probably crash. Even worse, if the attacker forms a clever line, it can rewrite the return address with specific bytes forming a valid address.

Should the attacker choose to redirect the return address directly into the buffer being overrun, he can transmit the executable code directly in this buffer. Such code is often calledshellcode, because it is small and usually only opens a remote shell to work with.

Obviously, this is not only the flaw ingetsbut the feature of the language itself. The moral is never to usegetsand always to provide a way to check the bounds of the target memory block.


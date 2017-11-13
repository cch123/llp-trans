14.1.6 vprintf and Friends

Functions such asprintf,fprintf, etc., have special versions. Those acceptva\_listas their last arguments. Their names are prefixed with a letter v, for example,

```
int vprintf(const char *format, va_list ap);
```

They are being used inside custom functions which in their turn accept an arbitrary number of arguments.

Listing14-8shows an example.

Listing 14-8.vsprintf.c

```
#include <stdarg.h>
#include <stdio.h>

void logmsg( int client_id, const char* const str, ... ) {
    va_list args;
    char buffer[1024];
    char* bufptr = buffer;
    
va_start( args, str );

bufptr += sprintf(bufptr, "from client %d :", client_id );
vsprintf( bufptr, str, args );
fprintf( stderr, "%s", buffer );

va_end( args );
}

```




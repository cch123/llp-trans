14.1.6 vprintf and Friends

像 printf，fprintf 这类的函数都有特殊的版本。特殊版本接收 va\_list 作为它们的最后的参数。这些特殊版本的函数以字母 v 作为前缀，例如：

```
int vprintf(const char *format, va_list ap);
```

这些自定义函数使用 va\_list 来接收任意数量的参数。

列表 14-8 展示了这样的例子。

_**Listing 14-8**.vsprintf.c_

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




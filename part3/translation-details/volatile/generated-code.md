14.2.2 Generated Code

我们来研究一下列表 14-11 中的示例。

_**Listing 14-11**.volatile\_ex.c_

```
#include <stdio.h>

int main( int argc, char** argv ) {
    int ordinary = 0;
    volatile int vol = 4;
    ordinary++;
    vol++;
    printf( "%d\n", ordinary );
    printf( "%d\n", vol );
    return 0;
}
```

有两个变量：其中一个是 volatile 类型，另一个不是。两者都被加一，并用 printf 进行了打印。GCC 会生成下面这样的代码\(在 -O2 优化级别下\)，见列表 14-12 所示：

_**Listing 14-12**.volatile\_ex.asm_

    ; these are two arguments for `printf`
    mov    esi,0x1
    mov    edi,0x4005d4

    ; vol = 4
    mov    DWORD PTR [rsp+0xc],0x4

    ; vol ++
    mov    eax,DWORD PTR [rsp+0xc]
    add    eax,0x1
    mov    DWORD  PTR  [rsp+0xc],eax

    xor eax,eax


    ; printf( "%d\n", ordinary )
    ; the `ordinary` is not even created in stack frame
    ; its final precomputed value 1 was placed in `rsi` in the first line!
    call   4003e0 <printf@plt>

    ; the second argument is taken from memory, it is volatile!
    mov    esi,DWORD PTR [rsp+0xc]

    ; First argument is the address of "%d\n"
    mov    edi,0x4005d4
    xor    eax,eax

    ; printf( "%d\n", vol )
    call   4003e0 <printf@plt>
    xor eax,eax

可以看到，C 语言中 volatile 变量的内容确实是在每次读写的时候真的进行了操作。而常见的普通变量甚至没有被创建出来：编译器直接进行了计算，而把结果存到了 rsi 寄存器，直接作为调用的第二个参数传入。


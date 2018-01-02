16.1.6 Constant Propagation

相关的 GCC 选项：-fipa-cp, -fgcse, -fipa-cp-clone 等等。.  
如果编译器能证明一个变量在程序的某个位置一定是某个具体的值，那么编译器可以省去读取变量值的过程而直接把值放在具体的代码位置。 

如果你打开 -fipa-cp-clone 选项的话，某些时候编译器甚至会因为你调用函数的参数，而为你生成特殊版本的函数。列表 16-10 展示了一种场景，这种情况下编译器会生成一个特殊版本的 sum 函数，该函数只接收一个参数，并且另一个参数已经固定为 42。

_**Listing 16-10**.constant\_propagation.c_

```
__attribute__ ((noinline))
static int sum(int x, int y) { return x + y; }

int main( int argc, char** argv ) {
    return sum( 42, argc );
}
```

列表 16-11 是翻译后的汇编代码。

_**Listing 16-11**.constant\_propagation.asm_

```
00000000004004c0   <sum.constprop.0>:
4004c0:       8d 47 2a                     lea       eax,[rdi+0x2a]
4004c3:       c3                           ret
```

编译器有时候还会帮你计算复杂的表达式\(包括函数调用\)。列表 16-12 展示了这样的例子。

_**Listing 16-12**.cp\_fact.c_

```
#include <stdio.h>

int fact( int n ) {
    if (n == 0) return 1;
    else return n * fact( n-1 );
}

int main(void) {
    printf("%d\n", fact( 4 ) );
    return 0;
}
```

这个阶乘函数显然每次都会计算出一样的结果，因为该函数的输出不依赖于任何用户输入。GCC 在这点上足够聪明，能够预先计算出值，并抹掉函数调用，直接把结果 24 写到目标代码，像列表 16-13 中展示的一样。汇编指令 mov edx, 0x18 把 16 进制的 18，也就是 10 进制的 24 直接移动到 rdx 寄存器。

_**Listing 16-13**.cp\_fact.asm_

```
0000000000400450 <main>:
400450:  48 83 ec 08         sub      rsp,0x8
400454:  ba 18 00 00 00      mov      edx,0x18
400459:  be 44 07 40 00      mov      esi,0x400744
40045e:  bf 01 00 00 00      mov      edi,0x1
400463:  31 c0               xor      eax,eax
400465:  e8 c6 ff ff ff      call     400430 <__printf_chk@plt>
40046a:  31 c0               xor      eax,eax
40046c:  48 83 c4 08         add      rsp,0x8
400470:  c3                  ret
```

如何活用本小节结论？命名常量没有任何负面影响，常量的变量也不会。编译器能够尽其所能帮助你预计算出一些表达式和函数调用的结果，同时不影响任何输入参数，或者引发什么副作用 。

同一函数每次调用都传入不同的参数时，可能会导致函数为每种参数组合都创建一份拷贝，这种情况有违程序的局部性原则。如果你面临性能问题，这方面的问题也需要纳入考量。


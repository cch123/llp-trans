16.1.5 消除公共子表达式

和该话题相关的 GCC 选项： -fgcse 和其它包含 cse 这个字符串的选项。

如果两个表达式中有相同的部分，不会导致该部分被计算两次。也就是说对这种公共部分进行预计算，把结果保存在变量中，然后在两个表达式中使用该变量，从优化角度来讲，这样做没什么用处。

在列表 16-8 的例子中，子表达式 x\*x + 2\*x 就只会被计算一次，而不像直观的感受那样。

_**Listing 16-8**.common\_subexpression.c_

```
#include <stdio.h>

__attribute__ ((noinline))
    void test(int x) {
        printf("%d %d",
                x*x + 2*x + 1,
                x*x + 2*x - 1 );
    }

int main(int argc, char** argv) {
    test( argc );
    return 0;
}
```

列表 16-9 是编译后的机器码，可以作为我们上述结论的佐证，对 x\*x + 2\*x 这个公共表达式不会计算两次。

_**Listing 16-9**.common\_subexpression.asm_

```
0000000000400516 <test>:
; rsi = x + 2
400516:       8d 77 02                           lea       esi,[rdi+0x2]
400519:       31 c0                              xor       eax,eax
40051b:       0f af f7                           imul      esi,edi
; rsi = x*(x+2)
40051e:       bf b4 05 40 00                     mov       edi,0x4005b4
; rdx = rsi-1 = x*(x+2) - 1
400523:       8d 56 ff                           lea       edx,[rsi-0x1]
; rsi = rsi + 1 = x*(x+2) - 1
400526:       ff c6                              inc       esi
400528:       e9 b3 fe ff ff                     jmp       4003e0 <printf@plt>
```

我们怎么活用这套结论呢？不要害怕在你漂亮的公式中出现一样的子表达式，编译器会高效地对它们进行计算。尽量从代码的可读性角度去考虑这种问题。


16.1.4 尾递归

本话题相关的 GCC 选项是：-formit-frame-pointer 和 -foptimize-sibling-calls。

让我们先来研究一下列表 16-3 中的函数。

```
 __attribute__ (( noinline ))
      int factorial( int acc, int arg ) {
          if ( arg == 0 ) return acc;
          return factorial( acc * arg, arg-1 );
      }

int main(int argc, char** argv) { return factorial(1, argc); }
```

函数递归地调用自身，但是这种调用比较特殊。一旦这个调用完成了，函数也就立即返回了。

我们说函数是尾递归函数，只要该函数满足下列条件的其中之一：

* 返回一个不需要递归调用的值，例如，return 4；
* 递归调用函数自身，传入其它参数并立即返回结果，而不是在结果的基础上进行计算。例如，return factorial\(acc \* arg, arg-1\)；

当递归调用的结果被用来做计算了的话，那函数就不是尾递归函数。

列表 16-4 展示了一个非尾递归阶乘计算函数的例子。递归调用的结果在返回之前要乘以 arg，所以不是尾递归。

_**Listing 16-4**.factorial\_nontailrec.c_

```
__attribute__ (( noinline ))
     int factorial( int arg ) {

        if ( arg == 0 ) return acc;
        return arg * factorial( arg-1 );
     }

int main(int argc, char** argv) { return factorial(argc); }
```

在第二章的时候，我们学习了 Question 20，该问题提出了一个解决尾递归的方案。当函数要做的最后一件事情是调用其它函数并直接跟一个 return 语句时；我们可以执行一条 jmp 指令跳到这个函数的开头。也就是说，下面这种模式的一连串指令可以被优化：

```
  ; somewhere else:
        call f

  ...
  ...

  f:

     ...
     call g
     ret    ; 1
  g:

     ...
     ret    ; 2
```

ret 指令在列表中被标记了 1 和 2。

执行 call g 将会把返回地址推到栈上。该地址实际上就是第一个 ret 指令的地址。当 g 完成了它的执行过程之后，会执行第二个 ret 指令，这时会从栈上弹出返回地址，使我们回到第一个 ret 指令的位置。因此两个 ret 指令在控制权被交还给 f 之前会依次执行。为什么不直接返回 f 的调用方呢？为了达到这个目的的话，我们需要把 call g 换成 jmp g。这样 g 就永远不能返回函数 f 了，我们也不能往栈上推一个没什么用的返回地址。第二个 ret 本来是通过 call f 来获取返回地址，现在这种操作需要在其它位置来做以使我们直接返回了。

```
; somewhere else:
    call f

...
...

f:

   ...
   jmp g
g:

   ...
   ret      ; 2
```

当 g 和 f 是同一个函数时，就是很明显的尾递归。在没有优化的前提下，factorial\(5,1\) 将五次执行自身，并“污染”五次栈帧。最后一次调用完毕之后会依次执行五次 ret 以处理好所有的返回地址。

现代编译器都能够有效的处理尾递归，并且知道如何将尾递归处理为一个循环。在列表 16-5 中列出的汇编代码是 GCC 对尾递归的阶乘计算\(列表 16-3\)所产生的。

_**Listing 16-5**.factorial\_tailrec.asm_

```
00000000004004c6 <factorial>:
4004c6:  89 f8                      mov     eax,edi
4004c8:  85 f6                      test    esi,esi
4004ca:  74 07                      je      4004d3 <factorial+0xd>
4004cc:  0f af c6                   imul    eax,esi
4004cf:  ff ce                      dec     esi
4004d1:  eb f5                      jmp     4004c8 <factorial+0x2>
4004d3:  c3                         ret
```

可以看到，尾递归调用由两阶段组成。

* 将新的参数值弹出到寄存器。
* 跳到函数开始部分

循环显然要比递归快得多，因为后者需要额外的栈上空间\(这样可能在递归层数深时爆栈\)。那为什么我们不在写代码阶段就用循环来取代递归呢？

因为递归使我们能够以更连贯和优雅的方式来表达一些算法。如果我们将函数写成尾递归的形式，那这种递归不会影响到程序的性能。

列表 16-6 展示了一个典型的，通过索引访问链表的函数。

_**Listing 16-6**.tail\_rec\_example\_list.c_

```
#include <stdio.h>
#include <malloc.h>

struct llist {
    struct llist* next;
    int value;
}

struct llist* llist_at(
        struct llist* lst,
        size_t idx ) {
    if ( lst && idx ) return llist_at( lst->next, idx-1 );
    return lst;
}

struct llist* c( int value, struct llist* next) {
    struct llist* lst = malloc( sizeof(struct llist*) );
    lst->next = next;
    lst->value = value;
    return lst;
}

int main( void ) {
    struct llist* lst = c( 1, c( 2, c( 3, NULL )));
    printf("%d\n", llist_at( lst, 2 )->value );
    return 0;
}
```

使用 -Os 选项编译将会产生非递归的代码，如列表 16-7 所示。

_**Listing 16-7**.tail\_rec\_example\_list.asm_

```
0000000000400596  <llist_at>:
400596:       48 89 f8                mov      rax,rdi
400599:       48 85 f6                test     rsi,rsi
40059c:       74 0d                   je       4005ab <llist_at+0x15>
40059e:       48 85 c0                test     rax,rax
4005a1:       74 08                   je       4005ab <llist_at+0x15>
4005a3:       48 ff ce                dec      rsi
4005a6:       48 8b 00                mov      rax,QWORD PTR [rax]
4005a9:       eb ee                   jmp      400599 <llist_at+0x3>
4005ab:       c3                      ret
```

如何活用本节知识？不要害怕使用尾递归，因为尾递归能够使代码更为可读且不会带来任何性能损失。


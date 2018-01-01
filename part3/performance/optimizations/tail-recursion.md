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

We say that the function istail recursiveif the function either

•Returns a value that does not involve a recursive call, for example,return 4;.

•Launches itself recursively with other arguments and returns the result immediately, without performing further computations with it. For example,return factorial  
 \( acc \* arg, arg-1 \);.

A function is not tail recursive when the recursive call result is then used in computations.

Listing16-4shows an example of a non-tail-recursive factorial computation. The result of a recursive call is multiplied byargbefore being returned, hence no tail recursion.

_**Listing 16-4**.factorial\_nontailrec.c_

```
__attribute__ (( noinline ))
     int factorial( int arg ) {

        if ( arg == 0 ) return acc;
        return arg * factorial( arg-1 );
     }

int main(int argc, char** argv) { return factorial(argc); }
```

In Chapter2, we studied Question 20, which proposes a solution in the spirit of tail recursion. When the last thing a function does is call other function, which is immediately followed by the return, we can perform a jump to the said function start. In other words, the following pattern of instructions can be a subject to optimization:

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

The ret instruction in this listing are marked as the first and the second one.

Executing call g will place the return address into the stack. This is the address of the firstretinstruction. Whengcompletes its execution, it executes the secondretinstruction, which pops the return address, leaving us at the firstret. Thus, tworetinstructions will be executed in a row before the control passes to the function that calledf. However, why not return to the caller offimmediately? To do that, we replace call g with jmp g. Now g we will never return to functionf, nor will we push a useless return address into the stack. The secondretwill pick up the return address fromcall f, which should have happened somewhere, and return us directly there.

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

Whengandfare the same function, it is exactly the case of tail recursion. When not optimized,factorial\(5, 1\)will launch itself five times, polluting the stack with five stack frames. The last call will end executingretfive times in a row in order to get rid of all return addresses.

Modern compilers are usually aware of tail recursive calls and know how to optimize tail recursion into a cycle. The assembly listing produced by GCC for the tail recursive factorial \(Listing16-3\) is shown in Listing16-5.

Listing 16-5.factorial\_tailrec.asm

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

As we see, a tail recursive call consists of two stages.

Populate registers with new argument values.

Jump to the function start.

Cycles are faster than recursion because the latter needs additional space in the stack \(which might lead to stack overflow as well\). Why not always stick with cycles then?

Recursion often allows us to express some algorithms in a more coherent and elegant way. If we can write a function so that it becomes tail recursive as well, that recursion will not impact the performance.

Listing16-6shows an exemplary function accessing a linked list element by index.

Listing 16-6.tail\_rec\_example\_list.c

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

How do we use it?Never be afraid to use tail recursion if it makes the code more readable for it brings no performance penalty.


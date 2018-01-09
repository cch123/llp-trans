17.3 Execution Order

When we started to study the C abstract machine, we got used to thinking that the sequence of C statements corresponds to the actions performed by the compiled machine instructions—naturally, in the same order. Now it is time to dive into the more pragmatic details of why it is not really the case.

We tend to describe algorithms in a way that is easier to understand, and this is almost always a good thing. However, the order given by the programmer is not always optimal performance-wise.

For example, the compiler might want to improvelocalitywithout changing the code semantics. Listing17-1shows an example.

Listing 17-1.ex\_locality\_src.c

```
char x[1000000], y[1000000];
...
x[4] = y[4];
x[10004] = y[10004];
x[5] = y[5];
x[10005] = y[10005];


```

列表 17 展示了一种可能的翻译结构。

_**Listing 17-2**.ex\_locality\_asm1.asm_

```
mov al,[rsi + 4]
mov [rdi+4],al

mov al,[rsi + 10004]
mov [rdi+10004],al

mov al,[rsi + 5]
mov [rdi+5],al

mov al,[rsi + 10005]
mov [rdi+10005],al
```

However, it is evident, that this code could be rewritten to ensure better locality; that is, first assignx\[4\]andx\[5\], then assignx\[10004\]andx\[10005\], as shown in Listing17-3.

Listing 17-3.ex\_locality\_asm2.asm

```
mov al,[rsi + 4]
mov [rdi+4],al
mov al,[rsi + 5]
mov [rdi+5],al


mov al,[rsi + 10004]
mov [rdi+10004],al

mov al,[rsi + 10005]
mov [rdi+10005],al
```

The effects of these two instruction sequences are similarif the abstract machine only considers one CPU: given any initial machine state, the resulting state after their executions will be the same. The second translation result often performs faster, so the compiler might prefer it. This is the simple case ofmemory reordering, a situation in which the memory accesses are reordered comparing to the source code.

For single thread applications, which are executed “really sequentially,” we can often expect the order of operations to be irrelevant as long as the observable behavior will be unchanged. This freedom ends as soon as we start communicating between threads.

Most inexperienced programmers do not think much about it because they limit themselves with single-threaded programming. In these days, we can no longer afford not to think about parallelism because of how pervasive it is and how it is often the only thing that can really boost the program performance. So, in this chapter, we are going to talk about memory reorderings and how to set them up correctly.


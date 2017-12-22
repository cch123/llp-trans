16.1.10 对代码中的读写操作分组

硬件对于没有交叉的连续读和连续写操作是支持得最好的。正因如此，在列表 16-19 中的代码一般都会比列表 16-20 中相似的代码要慢。后者的读写操作没有交叉，都是顺序的。

Listing 16-19.rwgroup\_bad.asm

```
mov rax,[rsi]
mov [rdi],rax
mov rax,[rsi+8]
mov [edi+4],eax
mov rax,[rsi+16]
mov [rdi+16],rax
mov rax,[esi+24]
mov [rdi+24],eax
```

Listing 16-20.rwgroup\_good.asm

```
mov rax, [rsi]
mov rbx, [rsi+8]
mov rcx, [rsi+16]
mov rdx, [rsi+24]
mov [rdi], rax
mov [rdi+8], rbx
mov [rdi+16], rcx
mov [rdi+24], rdx
```




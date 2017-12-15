16.1.10 Grouping Reads and Writes in Code

Hardware operates better with sequences of reads and writes which are not interleaved. For this reason, the code shown in Listing16-19is usually slower than its counterpart shown in Listing16-20. The latter has sequences of sequential reads and writes grouped rather than interleaved.

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




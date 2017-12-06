16.1.3 Omit Stack Frame Pointer

相关的 GCC 选项：-formit-frame-pointer。  
有时我们并不需要存储 rbp 的旧值并将其初始化为新值。这种不需要的场景具体是指：

* 下一次函数调用没有局部变量。
* 局部变量足够塞得进 red zone **并且 **函数没有调用另外的函数。

然而这样做也有负面影响：执行期间只保有程序的部分状态。这样在展开调用栈和获取局部变量时会比较麻烦，因为我们没法通过 rbp 来知道栈帧到底是从哪里开始的了。

特别是程序 crash 的时候我们需要 dump 现场来进行分析的时候，上面这种做法会导致很大的麻烦。这时的 dump 内容可能被重度优化而缺少了必要的调试信息。

从性能角度来说这种优化一般情况下都是微不足道的，参见\[26\]。

列表 16-1 的代码展示了在调用 unwind 时如何展开调用栈并显示所有函数的栈指针地址。用 -O0 选项来编译它以关闭优化。

_**Listing 16-1**.stack\_unwind.c_

```
void unwind();

void f( int count ) {

    if ( count ) f( count-1 ); else unwind();
}
int main(void) {
    f( 10 ); return 0;
}
```

列表 16-2 展示了一个例子。

_**Listing 16-2**.stack\_unwind.asm_

```
extern printf
global unwind

section .rodata
format : db "%x ", 10, 0

section .code
unwind:
push rbx

; while (rbx != 0) {
    ; print rbx; rbx = [rbx];
    ;}
    mov rbx, rbp
    .loop:
    test rbx, rbx
    jz .end
    mov rdi, format
    mov rsi, rbx
    call printf
    mov rbx, [rbx]
    jmp .loop

    .end:
    pop rbx
    ret
```

我们怎么使用它呢？在你的代码中包含一大堆无法内联的函数调用时，使用这段代码作为你最后的优化手段。


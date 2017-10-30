2.3.2 相对寻址

下面的例子展示了如何用比立即寻址更复杂的方式来访问内存。

_**Listing 2-6**.相对寻址:print\_rax.asm_

```
lea rsi, [codes + rax]
```

方括号表示**间接寻址；**地址被写到方括号里。

* mov rsi, rax --- 把 rax 的值拷贝到 rsi 中
* mov rsi, \[rax\] --- 是把内存内容 \(8 个顺序字节\) 拷贝到 rsi，内存的地址是从 rax 中取得的。我们怎么知道要拷贝的就是 8 个字节呢？像我们已经知道的 mov 的两个操作数字节数是相同大小，因为 rsi 是 8 个字节。知道了这些事实，汇编器也就可以推断需要拷贝 8 个字节了。

lea 和 mov 这两个指令有一点微妙的差别，lea 表示加载有效地址 "load effective address"。

lea 这个指令允许你计算一个内存单元的地址并把值存在其它地方。这个指令并非不重要，因为有一些 tricky 的寻址模式\(马上就会看到\)：例如，地址可能是几个操作数的求和值。

列表 2-7 提供了一个 lea 和 mov 的结果的演示

_**Listing 2-7**.lea\_vs\_mov.asm_

```
; rsi <- address of label 'codes', a number
mov rsi, codes

; rsi <- memory contents starting at 'codes' address
; 8 consecutive bytes are taken because rsi is 8 bytes long
mov rsi, [codes]

; rsi <- address of 'codes'
; in this case it is equivalent of mov rsi, codes
; in general the address can contain several components
lea rsi, [codes]

; rsi <- memory contents starting at (codes+rax)
mov rsi, [codes + rax]

; rsi <- codes + rax
; equivalent of combination:
; -- mov rsi, codes
; -- add rsi, rax
; Can't do it with a single mov!
lea rsi, [codes + rax]
```




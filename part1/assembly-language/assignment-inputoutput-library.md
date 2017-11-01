2.7 作业：输入输出库

在我们开始做一些看起来比较酷的东西之前，先来确保我们不需要一遍遍地写一些重复的基本函数。现在来讲，我们甚至还没法接收键盘键入的内容，暂时比较蛋痛。所以让我们先来为自己建立一个能接收输入和输出函数的库吧。

首先你需要阅读一下 Intel 的文档\[15\]，仔细阅读下面这些指令的说明\(请记住，指令的详细描述都在第二卷中\)：

* xor

* jmp,ja, 和类似指令

* cmp

* mov

* inc,dec

* add,imul,mul,sub,idiv,div

* neg

* call,ret

* push,pop

我们需要对上面这些指令非常了解，因为是我们现在需要的核心指令。你可能在翻文档的时候已经发现了，Intel 64 支持成千上万的指令。当然，我们并不需要在这里钻牛角尖。我们用目前学到的指令和系统调用已经可以应付目前大部分的工作。

此外还需要阅读 read 这个系统调用的文档。该系统调用编号为 0；类似的还有 write。如果感觉读不太懂，可以参考附录 C。

编辑我们自己的 lib.inc，提供一些定义好的函数，而不是像 `xor rax, rax` 这样的指令。表 2-2，列出了我们需要的函数的语义。推荐按照表格给出的顺序依次实现这些函数，因为可能在实现后面的函数的时候你发现前面写的代码可以重用。

_**Table 2-2**.Input/Output Library Functions_

| Function | Definition |
| :--- | :--- |
| exit | Accepts an exit code and terminates current process. |
| string\_length | Accepts a pointer to a string and returns its length. |
| print\_string | Accepts a pointer to a null-terminated string and prints it to stdout. |
| print\_char | Accepts a character code directly as its first argument and prints it to stdout. |
| print\_newline | Prints a character with code 0xA. |
| print\_uint | Outputs an unsigned 8-byte integer in decimal format.We suggest you create a buffer on the stack6 and store the division results there. Each time you divide the last value by 10 and store the corresponding digit inside the buffer. Do not forget, that you should transform each digit into its ASCII code \(e.g., 0x04 becomes 0x34\). |
| print\_int | Output a signed 8-byte integer in decimal format. |
| read_char | Read one character from stdin and return it. If the end of input stream occurs, return 0. |
| read_word | Accepts a buffer address and size as arguments. Reads next word from stdin (skipping whitespaces7 into buffer). Stops and returns 0 if word is too big for the buffer specified; otherwise returns a buffer address. This function should null-terminate the accepted string. |
| parse_uint | Accepts a null-terminated string and tries to parse an unsigned number from its start. Returns the number parsed in rax, its characters count in rdx. |
| parse_int | Accepts a null-terminated string and tries to parse a signed number from its start. Returns the number parsed in rax; its characters count in rdx (including sign if any). No spaces between sign and digits are allowed. |
| string_equals | Accepts two pointers to strings and compares them. Returns 1 if they are equal, otherwise 0. |
| string_copy | Accepts a pointer to a string, a pointer to a buffer, and buffer’s length. Copies string to the destination. The destination address is returned if the string fits the buffer; otherwise zero is returned. |

用 test.py 来自动化测试代码正确性。只要 run 一发脚本，剩下的脚本会帮你搞定。

记住，一个包含 n 个字符的的字符串需要 n + 1 个字节来存储，因为字符串结束是一个 null 结束符。

阅读附录 A 来获取寄存器和内存状态的单步跟踪调试技巧。


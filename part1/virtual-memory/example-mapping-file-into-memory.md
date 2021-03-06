4.9 示例：映射文件到内存

我们现在需要另一个叫 open 的系统调用。该系统调用使用名字打开文件，并获取到其描述符。

参见表 4-2 来获取细节：

_**Table 4-2**.open 系统调用_

| REGISTER | VALUE | MEANING |
| :--- | :--- | :--- |
| rax | 2 | 系统调用号 |
| rdi | 文件名 | 内容是指向 null 结尾字符串的指针 |
| rsi | flags | 权限标记位的组合\(read only，write only，或者读写均可\) |
| rdx | mode | 如果用 open 来创建文件，rdx 会持有该文件的权限 |

文件映射到内存需要三个简单步骤：

* 使用 open 系统调用打开文件，描述符会被放入 rax 寄存器。
* 调用 mmap，并传入相应的参数。其中的一个参数就是步骤一获取到的文件描述符。
* 使用我们在第二章写的 `print_string` 子过程来打印内容。为了简洁我们暂时省略关闭和错误检查过程。




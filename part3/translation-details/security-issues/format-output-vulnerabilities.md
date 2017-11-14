14.7.3 Format Output Vulnerabilities

Format output functions can be a source of very nasty bugs. There are several such functions in standard library; Table14-1shows them.

Table 14-1.String Format Functions

| Function | Description |
| :--- | :--- |
| printf | 输出格式化后的字符串 |
| fprintf | 将 printf 的结果写入到文件 |
| sprintf | 打印到一个字符串 |
| snprintf | 打印到字符串的同时检查长度 |
| vfprintf | 把 va\_arg 结构打印到文件 |
| vprintf | 把 va\_arg 打印到 stdout |
| vsprintf | 把 va\_arg 打印到字符串 |
| vsnprintf | 把 va\_arg 打印到字符串同时检查长度 |

Listing14-26shows an example. Suppose that the user inputs less than 100 symbols. Can you crash this program or produce other interesting effects?

Listing 14-26.printf\_vuln.c

```
#include <stdio.h>
int main(void) {
    char buffer[1024];
    gets(buffer);
    printf( buffer );
    return 0;
}
```

The vulnerability does not come fromgetsusage but from usage of the format string taken from the user. The user can provide a string that contains format specifiers, which will lead to an interesting behavior. We will mention several potentially unwanted types of behavior.

* •The"%x"specifiers and its likes can be used to view the stack contents. First 5"%x"will take arguments from registers \(rdiis already occupied with the format string address\), then the following ones will show the stack contents. Let’s compile the example shown in Listing14-26and see its reaction on an input"%x %x %x %x %x %x %x %x %x %x %x". 

```
> %x %x %x %x %x %x %x %x %x %x
b1b6701d b19467b0 fbad2088 b1b6701e 0 25207825 20782520 78252078 25207825

```



As we see, it actually gave us four numbers that share a certain informal similarity, a 0 and two more numbers. Our hypothesis is that the last two numbers are taken from the stack already.

Getting intogdband exploring the memory near the stack top right afterprintfcall we are going to get results that prove our point. Listing14-27shows the output.

Listing 14-27.gdb\_printf

```
(gdb) x/10 $rsp
0x7fffffffdfe0: 0x25207825 0x78252078   0x20782520 0x25207825
0x7fffffffdff0: 0x78252078 0x20782520   0x25207825 0x00000078
0x7fffffffe000: 0x00000000 0x00000000
```

* •The"%s"format specifier is used to print strings. As a string is defined by the address of its start, this means addressing memory by a pointer. So, if no valid pointer is given, the invalid pointer will be dereferenced.

---

■Question 266 What will be the result of launching the code shown in listing14-26on input"%s %s %s %s %s"?

---

* •The"%n"format specifier is a bit exotic but still harmful. It allows one to write an integer into memory. Theprintffunction accepts apointerto an integer which will be rewritten with an amount of symbols written so far \(before"%n"occurs\). Listing14-28shows an example of its usage.

Listing 14-28.printf\_n.c

\#include &lt;stdio.h&gt;



```
int main(void) {
    int count;    
    printf( "hello%n world\n", &count);    
    printf( "%d\n", count );    
    return 0;
}
```

This will output 5, because there were five symbols output before"%n". This is not a trivial string length because there can be other format specifiers before, which will result in an output of variable length \(e.g., printing an integer can emit seven or ten symbols\). Listing14-29shows an example.

Listing 14-29.printf\_n\_ex.c

```
int x;
printf("%d %n", 10, &x);  /* x = 3 */
printf("%d %n", 200, &x); /* x = 4 */
```

To avoid that, do not use the string accepted from the user as a format string. You can always writeprintf\("%s", buffer\), which is safe as long as the buffer is not NULL and is a valid null-terminated string. Do not forget about such functions asputsoffputs, which are not only faster but also safer.


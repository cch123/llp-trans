14.7.2 return-to-libc

As we have already elaborated, the malevolent user can rewrite the return address if the program allows him to overrun the stack buffer. The return-to-libc attack is performed when the return address is the address  
 of a function in the standard C library. One function is of a particular interest,int system\(const char\* command\). This function allows you to execute an arbitrary shell command. What’s even worse, it will be executed with the same privileges as the attacked program.

When the current function terminates by executing theretcommand, we will start executing the function from libc. It is yet a question, how do we form a valid argument for it?

In the presence ofASLR\(address space layout randomization\), doing this attack is nontrivial \(but still possible\).


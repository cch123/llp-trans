14.8.2 Address Space Layout Randomization

Loading each program section to a random place in an address space makes it nearly impossible to guess a correct return address to perform an intelligent jump. Most commonly used operating systems support it; however, that feature should be enabled during the compilation. In this case, the information about ASLR support will be stored in the executable file itself, which will force the loader to perform a correct relocation.


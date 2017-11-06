4.9 Example: Mapping File into Memory

We need another system call, namely,open. It is used to open a file by name and to acquire itsdescriptor.

See Table4-2for details.



Table 4-2.openSystem Call

Mapping file in memory is done in three simple steps:

Open file usingopensystem call.raxwill hold file descriptor.

Call mmap with relevant arguments. One of them will be the file descriptor, acquired at step 1.

Useprint\_stringroutine we have created in Chapter2. For the sake of brevity we omit file closing and error checks.


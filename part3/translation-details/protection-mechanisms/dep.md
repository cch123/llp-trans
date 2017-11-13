14.8.3 DEP

We have already discussedData Execution Preventionin Chapter4This technology protects some pages from executing instructions stored on these pages. To enable it, programs should be also compiled with support turned on.

The sad fact is that it does not work well with programs that use just-in-time compilation, which forms executable code during the program execution itself. This is not as rare as it might seem; for example, virtually all browsers are using JavaScript engines which support just-in-time compilation.


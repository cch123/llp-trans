16.1 优化

In this section we want to discuss the most important optimizations that happen during the translation process. They are crucial to understanding how to write quality code. Why? A common type of decision making in programming is balancing between code readability and performance. Knowing optimizations  
 is necessary in order to make good decisions. Otherwise, when choosing between two versions of code, we might choose a less readable one because it “looks” like it performs fewer actions. In reality, however, both versions will be optimized to exactly the same sequences of assembly instructions. In this case, we just made a less readable code for no benefit at all.

---

■Note In the listings presented in this section we will often use an\_\_attribute\_\_ \(\(noinline\)\)GCC directive. Applying it to a function definition suppresses inlining for the said function. Exemplary functions are often small, which encourages compilers to inline them, which we do not want to better show various optimization effects.

Alternatively, we could have compiled the examples with-fno-inlineoption.

---




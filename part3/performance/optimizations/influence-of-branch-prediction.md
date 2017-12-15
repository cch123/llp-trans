16.1.8 Influence of Branch Prediction

On the microcode level the actions performed by the CPU \(central processing unit\) are even more primitive than the machine instructions; they are also reordered to better use all CPU resources.

Branch prediction is a hardware mechanism that is aimed at improving program execution speed. When the CPU sees a conditional branch instruction \(such asjg\), it can

* Start executing both branches simultaneously; or

* Guess which branch will be executed and start executing it.

It happens when the computation result \(e.g., theGFflag value injg \[rax\]\) on which this jump destination depends is not yet ready, so we start executing code speculatively to avoid wasting time.

The branch prediction unit can fail by issuing amisprediction. In this case, once the computation is completed, the CPU will do an additional work of reverting the changes made by instructions from the wrong branch. It is slow and has a real impact on program performance, but mispredictions are relatively rare.

The exact branch prediction logic depends on the CPU model. In general, two types of prediction exist \[6\]:staticanddynamic.

* If the CPU has no information about a jump \(when it is executed for the first time\), a static algorithm is used. A possible simple algorithm is as follows:

– If this is a forward jump, we assume that it happens.

– If it is a backward jump, we assume that it does not happen.

It makes sense because the jumps used to implement loops are more likely to happen than not.

* If a jump has already happened in the past, the CPU can use more complex algorithms. For example, we can use a ring buffer, which stores information about whether the jump occurred or not. In other words, it stores the history of jumps. When this approach is used, small loops of length dividing the buffer length are good for prediction.



The best source of relevant information with regard to the exact CPU model can be found in \[16\]. Unfortunately, most information about the CPU innards is not disclosed to public.

How do we use it?When using if-then-else or switch start with the most likely cases. You can also use special hints such as\_\_builtin\_expectGCC directives, which are implemented as special instruction prefixes for jump instructions \(see \[6\]\).


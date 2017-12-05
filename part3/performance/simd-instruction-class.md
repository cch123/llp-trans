16.3 SIMD Instruction Class

The von Neumann computational model is sequential by its nature. It does not presume that some operations can be executed in parallel. However, in time it became apparent that executing actions in parallel is necessary to achieve better performance. It is possible when the computations are independent from one another. For example, in order to sum 1 million integers we can calculate the sum of the chunks of 100,000 numbers on ten processors and then sum up the results. It is a typical kind of task that is solved well by themap-reducetechnique \[5\].

We can implement parallel execution in two ways.

* Parallel execution of several sequences of instructions. That is achievable by introducing additional processor cores. We will discuss the multithreaded programming that makes use of multiple cores in Chapter17.

* Parallel execution of actions that are needed to complete asingle instruction. In this case we can have instructions which invoke multiple independent computations spanning the different parts of processor circuits, which are exploitable in parallel.  
   To implement such instructions, the CPU has to include several ALUs to have actual performance gains, but it does not need to be able to execute multiple instructions truly simultaneously. These instructions are calledSIMD\(Single Instruction, Multiple Data\) instructions.

In this section we are going to overview the CPU extensions calledSSE \(Streaming SIMD Extensions\)and its newer analogueAVX \(Advanced Vector Extensions\).

Contrary to SIMD instructions, most instructions we have studied yet are of the type SISD \(Single Instruction, Single Data\).






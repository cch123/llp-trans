CHAPTER 16 Performance

In this chapter we will study how to write faster code. In order to do that, we will look into SSE \(Streaming SIMD Extensions\) instructions, study compiler optimizations, and hardware cache functioning.

Note that this chapter is a mere introduction to the topic and will not make you an expert in optimization.

There is no silver bullet technique to magically make everything fast. Hardware has become so complex that even an educated guess about the code that is slowing down program execution might fail. Testing and profiling should always be performed, and the performance should be measured in a reproducible way. It means that everything about the environment should be described in such detail that anyone would be able to replicate the conditions of the experiment and receive similar results.


Chapter 17 Multithreading

In this chapter we will explore the multithreading capabilities provided by the C language. Multithreading is a topic for a book on its own, so we will concentrate on the language features and relevant properties of the abstract machine rather than good practices and program architecture-related topics.

Until C11, the support of the multithreading was external to the language itself, via libraries and nonstandard tricks. A part of it \(atomics\) is now implemented in many compilers and provides a standard-compliant way of writing multithreaded applications. Unfortunately, to this day, the support of threading itself is not implemented in most toolchains, so we are going to use the librarypthreadsto write down code examples. We will still use the standard-compliant atomics.

This chapter is by no means an exhaustive guide to multithreaded programming, which is a beast worth writing a dedicated book about, but it will introduce the most important concepts and relevant language features. If you want to become proficient in it, we recommend lots of practice, specialized articles, books such as \[34\], and code reviews from your more experienced colleagues.


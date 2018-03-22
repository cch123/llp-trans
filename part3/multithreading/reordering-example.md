17.5 Reordering Example

Listing17-4 shows an exemplary situation when memory reordering can give us a bad day. Here two threads are executing the statements contained in functionsthread1andthread2, respectively.

Listing 17-4.mem\_reorder\_sample.c

```
int x = 0;
int y = 0;

void thread1(void) {
    x = 1;
    print(y);
}

void thread2(void) {
    y = 1;
    print(x);
}
```

Both threads share variablesxandy. One of them performs a store intoxand then loads the value ofy, while the other one does the same, but withyandxinstead.

We are interested in two types of memory accesses:loadandstore. In our examples, we will often omit all other actions for simplicity.

As these instructions are completely independent \(they operate on different data\), they can be reordered inside each thread without changing observable behavior, giving us four options:store + loadorload + storefor each of two threads. This is what a compiler can do for its own reasons. For each optionsixpossible execution orders exist. They depict how both threads advance in time relative to one another.

We show them as sequences of 1 and 2; if the first thread made a step, we write 1; otherwise the second one made a step.

1.1-1-2-2

2.1-2-1-2

3.2-1-1-2

4.2-1-2-1

5.2-2-1-1

6.1-2-2-1

For example, 1-1-2-2 means that the first process has executed two steps, and then the second process did the same. Each sequence corresponds to four different scenarios. For example, the sequence 1-2-1-2 encodes one of the traces, shown in Table17-1:

Table 17-1.Possible Instruction Execution Sequences If Processes Take Turns as 1-2-1-2

注意，这里有一个表格

If we observe these possible traces for each execution order, we will come up with24scenarios \(some of which will be equivalent\). As you see, even for the small examples these numbers can be large enough.

We do not need all these possible traces anyway; we are interested in the position ofloadrelatively tostorefor each variable. Even in Table17-1many possible combinations are present: bothxandycan be stored, then loaded, or loaded, then stored. Obviously, the result ofloadis dependent on whether there was astorebefore.

Were reorderings not in the game, we would be limited: any of the two specifiedloadsshould have been preceded by a store because so it is in the source code; scheduling instructions in a different manner cannot change that. However, as the reorderings are present, we can sometimes achieve an interesting outcome: if both of these threads have their instructions reordered, we come to a situation shown in Listing17-5.



Listing 17-5.mem\_reorder\_sample\_happened.c

```
int x = 0;
int y = 0;

void thread1(void) {
    print(y);
    x = 1;
}

void thread2(void) {
    print(x);
    y = 1;
}
```

If the strategy 1-2-\*-\* \(where \* denotes any of the threads\) is chosen, we executeload xandload yfirst, which will make them appear to equal to 0 for everyone who uses theseloads’ results

It is indeed possible in casecompilerreordered these operations. But even if they are controlled well or disabled, the memory reorderings, performed by CPU, still can produce such an effect.

This example demonstrates that the outcome of such a program is highly unpredictable. Later we are going to study how to limit reorderings by the compiler and by CPU; we will also provide a code to demonstrate this reordering in the hardware.


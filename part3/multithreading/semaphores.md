17.9 信号量

信号量是一种共享的 int 类型变量，允许以下三种操作：

* 使用参数 N 来将其值初始化为 N。
* Wait\(enter\)。如果信号量非零的话，对变量减。否则等待其他人对该变量进行加之后，然后再减。
* Post\(leave\)。增加信号量的值。

显然这个值不能以普通的方式直接访问，且不能被减到 0 以下。

信号量并不是 pthreads 标准的一部分，我们使用 POSIX 标准提供的 interface 来操作信号量。因此，操作信号量的代码在编译时需要带上 pthread flag。

大多数 UNIX-like 的操作系统都实现了标准的 pthreads 特性和信号量。使用信号量来进行线程间的同步操作很常见。

列表 17-17 展示了一个信号量的使用例子。

_**Listing 17-17**.semaphore\_mwe.c_

```
#include <semaphore.h>
#include <inttypes.h>
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>

sem_t sem;

uint64_t counter1 = 0;
uint64_t counter2 = 0;

pthread_t t1, t2, t3;

void* t1_impl( void* _ ) {
    while( counter1 < 10000000 ) counter1++;
    sem_post( &sem );
    return NULL;
}

void* t2_impl( void* _ ) {
    while( counter2 < 20000000 ) counter2++;
    sem_post( &sem );
    return NULL;
}

void* t3_impl( void* _ ) {
    sem_wait( &sem );
    sem_wait( &sem );
    printf("End: counter1 = %" PRIu64 " counter2 = %" PRIu64 "\n",
            counter1, counter2 );
    return NULL;
}

int main(void) {
    sem_init( &sem, 0, 0 );
    pthread_create( &t3, NULL, t3_impl, NULL );
    sleep( 1 );
    pthread_create( &t1, NULL, t1_impl, NULL );
    pthread_create( &t2, NULL, t2_impl, NULL );
    sem_destroy( &sem );
    pthread_exit( NULL );
    return 0;
}
```

sem\_init 函数会初始化信号量。第二个参数是一个 flag：0 对应进程本地信号量\(该信号量会被不同的线程所使用\)，非零变量初始化一个对多个进程可见的信号量。第三个参数负责初始化初始的信号量的值。使用 `sem destroy `函数负责删除信号量。这个例子里，创建了两个计数器和三个线程。t1 和 t2 线程交错执行，将对应的计数器增加到 1000000 和 2000000，然后用 `sem post` 将信号量值增加。t3 通过减信号量的值两次对其自身上锁。之后信号量被其它线程加两次之后，t3 将其计数器的值打印到 stdout。

`pthread_exit `调用保证主线程等待其它线程都工作完之后再安全退出。

信号量对于下面类型的工作比较擅长：

* 禁止 n 个以上的进程同时操作代码片段。
* 使一个线程等待另一个线程完成特定操作之后，然后在此基础上执行自己的工作。
* 限定并行工作的线程数量。线程数太多也可能影响程序性能。

有人认为信号量和 mutex 一样只有两种状态，这是错误的。mutex 的 lock 和 unlock 两种状态都只能被同一个线程操作，但信号量可以被任意的线程增减。

在列表 17-18 中还有另外一个使用信号量的例子，两个线程同时开始循环叠代\(在循环体内他们等待另外的循环来完成自己的叠代\)。

对信号量的操作和编译器/硬件的内存屏障有些类似。

想要了解更多信号量的详情的话，可以参考下面这些方法的 man page：

* em\_close
* sem\_destroy
* sem\_getvalue
* sem\_init
* sem\_open
* sem\_post
* sem\_unlink
* sem\_wait

---

**■Question 364** 什么是命名信号量？为什么在进程退出的时候应该手动 unlink 掉这种信号量？

---




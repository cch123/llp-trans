17.8.3 管理线程

我们学习到的知识已经足够来支持我们并行地工作了。不过还缺少进行同步的手段，尽管我们能够将工作分发给线程，但却还不能真正使用其它线程的计算结果。

最简单的同步手段是线程 join。这个想法很简单：我们在一个 pthread 的实例上调用  `thread_join `就可以将当前线程置于等待状态了，该状态会一直持续到其它线程终止。列表 17-8 展示了一个例子：

_**Listing 17-8**.thread\_join\_mwe.c_

```
#include <pthread.h>
#include <unistd.h>
#include <stdio.h>
void* worker( void* param ) {
    for( int i = 0; i < 3; i++ ) {
        puts( (const char*) param );
        sleep(1);
    }
    return (void*)"done";
}

int main( void ) {
    pthread_t t;
    void* result;
    pthread_create( &t, NULL, worker, (void*) "I am a worker!" );
    pthread_join( t, &result );
    puts( (const char*) result );
    return 0;
}
```

thread\_join 接收两个参数：线程本身以及一个 void\* 类型的变量地址，该 void \* 变量会被初始化为线程的执行结果。

线程 join 的表现和一个 full barrier 差不多，因为我们不会在 join 之前执行任何本应稍后才执行的读写操作。

默认情况下，线程创建时都是可 join 状态的，但有可能会创建出 detached 线程。这种线程的好处是能够在线程终止的时候立刻释放其持有的资源。而可 join 的线程，则会一直等待 join 之后，才能释放其持有资源。创建 detached 线程的流程为：

* 创建一个 attribute 类型的实例 pthread\_attr\_t attr;
* 用 pthread\_attr\_init\(&attr\) 初始化它
* 调用 pthread\_attr\_setdetachstate\(&attr, PTHREAD\_CREATE\_DETACHED\)；然后
* 使用 pthread\_create 和 attr 作为其参数

当前线程能够通过调用 pthread\_detach\(\) 显式地从可 join 状态切换为 detached 状态。反过来的状态切换没有办法做到的。


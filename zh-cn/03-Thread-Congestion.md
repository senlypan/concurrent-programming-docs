# Thread Congestion in Java

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.concurrent.03-thread-congestion&left_color=blue&right_color=red)

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/thread-congestion.html  Update: 2022-02-24

## ⛔抱歉，本文暂无中文翻译，持续更新中
?> ❤️ 您也可以参与翻译，快来提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿参与吧~

Thread congestion can occur when two or more threads are trying to access the same, guarded data structure at the same time. By "guarded" I mean that the data structure is guarded using synchronized blocks or a concurrent data structure (Lock, BlockingQueue etc.) so that the data structure is thread safe. The resulting thread congestion means that the threads trying to access the shared data structure are spending a high amount of time waiting in line to access the data structure - wasting valuable execution time on waiting.

![03-Thread-Congestion.md#thread-congestion-1.png](http://tutorials.jenkov.com/images/java-concurrency/thread-congestion-1.png)

## Thread Congestion Tutorial Video

If you prefer video I have a video version of this thread congestion tutorial here: [Thread Congestion in Java](https://www.youtube.com/watch?v=DqpPRxCmxrM&list=PLL8woMHwr36EDxjUoCzboZjedsnhLP1j4&index=22%22)

![03-Thread-Congestion.md#thread-congestion-video-screenshot.png](http://tutorials.jenkov.com/images/java-concurrency/thread-congestion-video-screenshot.png)

## Thread Blocking Data Structures May Cause Thread Congestion

A data structure which blocks threads from accessing it - depending on what other threads are currently accessing it - may cause thread congestion. If more than one thread accesses such a data structure at the same time, one or more of the threads may be queued up waiting to access the data structure.

This queuing up is not visible in your code. The queuing up happens inside the Java VM. Therefore thread congestion is not so easy to spot simply by looking at your code. You may need profiling tools to detect thread congestion - or, you need to learn where to predict thread congestion can occur.

![03-Thread-Congestion.md#thread-congestion-2.png](http://tutorials.jenkov.com/images/java-concurrency/thread-congestion-2.png)

## A Blocked Thread Loses Execution Time

While a thread is blocked trying to execute a blocking data structure, it cannot do anything. Thus, while being blocked the thread loses possible execution time. The longer the thread is blocked, the more potential execution time it loses.

## The More Threads - The Higher Congestion

The more threads that attempt to access a shared, blocking data structure, the higher the risk is of thread congestion occurring, and the higher the congestion may be (the number of threads queued up waiting to access the data structure).

## Alleviating Thread Congestion

To alleviate thread congestion you must reduce the number of threads trying to access the blocking data structure at exactly the same time. There are several ways to do that.

### Multiple Data Structures

One way to alleviate thread congestion - at least around blocking queues - is to give each consuming thread its own queue, and have the producing thread distribute the objects (e.g. tasks) among these blocking queues. This way, only 2 threads are ever accessing each queue: The producing thread and the consuming thread.

![03-Thread-Congestion.md#thread-congestion-3.png](http://tutorials.jenkov.com/images/java-concurrency/thread-congestion-3.png)

### Non-blocking Concurrency Algorithms

Another way is to use non-blocking concurrency algorithms where the threads accessing the data structure are never blocked. Non-blocking concurrency algorithms and data structures can often waste less of the thread's potential execution time than blocking concurrency algorithm or data structure.

The End.

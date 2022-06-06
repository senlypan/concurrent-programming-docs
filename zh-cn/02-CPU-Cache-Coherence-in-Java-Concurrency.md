# CPU Cache Coherence in Java Concurrency

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.concurrent.02-cpu-cache-coherence-in-java-concurrency&left_color=blue&right_color=red)

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/cache-coherence-in-java-concurrency.html  Update: 2022-02-24

## ⛔抱歉，本文暂无中文翻译，持续更新中
?> ❤️ 您也可以参与翻译，快来提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿参与吧~

In some of the other tutorials in this Java Concurrency tutorial series you might have read, or heard me saying in a video, that when a Java thread writes to a volatile variable, or exits a synchronized block - that this flushes all variables visible to the thread from the CPU cache to main memory.

This is not actually what happens. In this short tutorial I will explain what really happens. I have made a video version of this tutorial too, if you prefer video:

[Java Concurrency + CPU Cache Coherence](https://www.youtube.com/watch?v=nNXkzDS6dOg&list=PLL8woMHwr36EDxjUoCzboZjedsnhLP1j4&index=7)

![02-CPU-Cache-Coherence-in-Java-Concurrency.md#java-concurrency-plus-cache-coherence-video-screenshot.png](http://tutorials.jenkov.com/images/java-concurrency/java-concurrency-plus-cache-coherence-video-screenshot.png)

What actually happens is, that all variables visible to the thread which are stored in CPU registers will be flushed to main RAM (main memory). On the way to main RAM the variables may be stored in the CPU cache. The CPU / motherboard then uses its cache coherence methods to make sure that all other CPUs caches can see the variables in the first CPUs cache.

The hardware may even choose not to flush the variables all the way to main memory but only keep it in the CPU cache - until the CPU cache storing the variables is needed for other data. At that time the CPU cache can then be flushed to main memory. However, for the code running on the CPU this is not visible. As long as it gets the data it requests from any given memory address, it doesn't matter if the returned data only exists in the CPU cache, or whether it is also exists in main RAM.

You don't have to worry about how CPU cache coherence works. There is of course a little performance hit for this CPU cache coherence, but it's better than writing the variables all the way down to main RAM and back up the other CPU caches.

Below is a diagram illustrating what I said above. The red, dashed arrow on the left represents my false statement from other tutorials - that variables were flushed from CPU cache to main RAM. The arrow on the right represents what actually happens - that variables are flushed from CPU registers to the CPU cache.

![02-CPU-Cache-Coherence-in-Java-Concurrency.md#cpu-cache-coherence-and-java-concurrency-1.png](http://tutorials.jenkov.com/images/java-concurrency/cpu-cache-coherence-and-java-concurrency-1.png)

The End.
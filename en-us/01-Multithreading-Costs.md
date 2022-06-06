# Multithreading Costs

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.concurrent.01-multithreading-costs-en&left_color=blue&right_color=red)

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/costs.html  Update: 2022-02-23

Going from a singlethreaded to a multithreaded application doesn't just provide benefits. It also has some costs. Don't just multithread-enable an application just because you can. You should have a well-founded expectation that the benefits gained by doing so are larger than the costs. When in doubt, try measuring the performance or responsiveness of the application, instead of just guessing.

## More complex design

Though some parts of a multithreaded applications are simpler than a singlethreaded application, other parts are more complex. Code executed by multiple threads accessing shared data need special attention. Thread interaction is far from always simple. Errors arising from incorrect thread synchronization can be very hard to detect, reproduce and fix.

## Context Switching Overhead

When a CPU switches from executing one thread to executing another, the CPU needs to save the local data, program pointer etc. of the current thread, and load the local data, program pointer etc. of the next thread to execute. This switch is called a "context switch". The CPU switches from executing in the context of one thread to executing in the context of another.

Context switching isn't cheap. You don't want to switch between threads more than necessary.

You can read more about context switching on Wikipedia:

http://en.wikipedia.org/wiki/Context_switch

## Increased Resource Consumption

A thread needs some resources from the computer in order to run. Besides CPU time a thread needs some memory to keep its local stack. It may also take up some resources inside the operating system needed to manage the thread. Try creating a program that creates 100 threads that does nothing but wait, and see how much memory the application takes when running.

The End.
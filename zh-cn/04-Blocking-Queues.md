# Blocking Queues

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/blocking-queues.html  Update: 2022-02-24

## ⛔抱歉，本文暂无中文翻译，持续更新中
?> ❤️ 您也可以参与翻译，快来提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿参与吧~

A blocking queue is a queue that blocks when you try to dequeue from it and the queue is empty, or if you try to enqueue items to it and the queue is already full. A thread trying to dequeue from an empty queue is blocked until some other thread inserts an item into the queue. A thread trying to enqueue an item in a full queue is blocked until some other thread makes space in the queue, either by dequeuing one or more items or clearing the queue completely.

Here is a diagram showing two threads cooperating via a blocking queue:

![04-Blocking-Queues.md#blocking-queue.png](http://tutorials.jenkov.com/images/java-concurrency-utils/blocking-queue.png)

Java 5 comes with blocking queue implementations in the java.util.concurrent package. You can read about that class in my java.util.concurrent.BlockingQueue tutorial. Even if Java 5 comes with a blocking queue implementation, it can be useful to know the theory behind their implementation.

## Blocking Queue Implementation

The implementation of a blocking queue looks similar to a Bounded Semaphore. Here is a simple implementation of a blocking queue:

```java
public class BlockingQueue {

  private List queue = new LinkedList();
  private int  limit = 10;

  public BlockingQueue(int limit){
    this.limit = limit;
  }


  public synchronized void enqueue(Object item)
  throws InterruptedException  {
    while(this.queue.size() == this.limit) {
      wait();
    }
    this.queue.add(item);
    if(this.queue.size() == 1) {
      notifyAll();
    }
  }


  public synchronized Object dequeue()
  throws InterruptedException{
    while(this.queue.size() == 0){
      wait();
    }
    if(this.queue.size() == this.limit){
      notifyAll();
    }

    return this.queue.remove(0);
  }

}
    
```

Notice how notifyAll() is only called from enqueue() and dequeue() if the queue size is equal to the size bounds (0 or limit). If the queue size is not equal to either bound when enqueue() or dequeue() is called, there can be no threads waiting to either enqueue or dequeue items.

The End.
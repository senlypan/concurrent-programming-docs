# Semaphores

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.concurrent.04-semaphores-en&left_color=blue&right_color=red)

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/semaphores.html  Update: 2022-02-24

A Semaphore is a thread synchronization construct that can be used either to send signals between threads to avoid missed signals, or to guard a critical section like you would with a lock. Java 5 comes with semaphore implementations in the java.util.concurrent package so you don't have to implement your own semaphores. Still, it can be useful to know the theory behind their implementation and use.

Java 5 comes with a built-in Semaphore so you don't have to implement your own. You can read more about it in the java.util.concurrent.Semaphore text, in my java.util.concurrent tutorial.

## Simple Semaphore

Here is a simple Semaphore implementation:

```java
public class Semaphore {
  private boolean signal = false;

  public synchronized void take() {
    this.signal = true;
    this.notify();
  }

  public synchronized void release() throws InterruptedException{
    while(!this.signal) wait();
    this.signal = false;
  }

}
```

The `take()` method sends a signal which is stored internally in the Semaphore. The `release()` method waits for a signal. When received the signal flag is cleared again, and the `release()` method exited.

Using a semaphore like this you can avoid missed signals. You will call `take()` instead of `notify()` and `release()` instead of `wait()`. If the call to `take()` happens before the call to `release()` the thread calling `release()` will still know that `take()` was called, because the signal is stored internally in the `signal` variable. This is not the case with `wait()` and `notify()`.

The names `take()` and `release()` may seem a bit odd when using a semaphore for signaling. The names origin from the use of semaphores as locks, as explained later in this text. In that case the names make more sense.

## Using Semaphores for Signaling

Here is a simplified example of two threads signaling each other using a Semaphore:

```java
Semaphore semaphore = new Semaphore();

SendingThread sender = new SendingThread(semaphore);

ReceivingThread receiver = new ReceivingThread(semaphore);

receiver.start();
sender.start();
```

```java
public class SendingThread {
  Semaphore semaphore = null;

  public SendingThread(Semaphore semaphore){
    this.semaphore = semaphore;
  }

  public void run(){
    while(true){
      //do something, then signal
      this.semaphore.take();

    }
  }
}
```

```java
public class RecevingThread {
  Semaphore semaphore = null;

  public ReceivingThread(Semaphore semaphore){
    this.semaphore = semaphore;
  }

  public void run(){
    while(true){
      this.semaphore.release();
      //receive signal, then do something...
    }
  }
}
```

## Counting Semaphore

The Semaphore implementation in the previous section does not count the number of signals sent to it by take() method calls. We can change the Semaphore to do so. This is called a counting semaphore. Here is a simple implementation of a counting semaphore:

```java
public class CountingSemaphore {
  private int signals = 0;

  public synchronized void take() {
    this.signals++;
    this.notify();
  }

  public synchronized void release() throws InterruptedException{
    while(this.signals == 0) wait();
    this.signals--;
  }

}
```

## Bounded Semaphore

The CoutingSemaphore has no upper bound on how many signals it can store. We can change the semaphore implementation to have an upper bound, like this:

```java
public class BoundedSemaphore {
  private int signals = 0;
  private int bound   = 0;

  public BoundedSemaphore(int upperBound){
    this.bound = upperBound;
  }

  public synchronized void take() throws InterruptedException{
    while(this.signals == bound) wait();
    this.signals++;
    this.notify();
  }

  public synchronized void release() throws InterruptedException{
    while(this.signals == 0) wait();
    this.signals--;
    this.notify();
  }
}
```

Notice how the `take()` method now blocks if the number of signals is equal to the upper bound. Not until a thread has called `release()` will the thread calling `take()` be allowed to deliver its signal, if the BoundedSemaphore has reached its upper signal limit.

## Using Semaphores as Locks

It is possible to use a bounded semaphore as a lock. To do so, set the upper bound to 1, and have the call to `take()` and `release()` guard the critical section. Here is an example:

```java
BoundedSemaphore semaphore = new BoundedSemaphore(1);

...

semaphore.take();

try{
  //critical section
} finally {
  semaphore.release();
}
```

In contrast to the signaling use case the methods` take()` and `release()` are now called by the same thread. Since only one thread is allowed to take the semaphore, all other threads calling `take()` will be blocked until `release()` is called. The call to `release()` will never block since there has always been a call to `take()` first.

You can also use a bounded semaphore to limit the number of threads allowed into a section of code. For instance, in the example above, what would happen if you set the limit of the `BoundedSemaphore` to 5? 5 threads would be allowed to enter the critical section at a time. You would have to make sure though, that the thread operations do not conflict for these 5 threads, or you application will fail.

The `relase()` method is called from inside a finally-block to make sure it is called even if an exception is thrown from the critical section.

The End.
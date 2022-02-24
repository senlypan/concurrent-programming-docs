# Nested Monitor Lockout

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/nested-monitor-lockout.html  Update: 2022-02-24

## ⛔抱歉，本文暂无中文翻译
?> ❤️ 您也可以参与翻译，快来提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿参与吧~

## How Nested Monitor Lockout Occurs

Nested monitor lockout is a problem similar to deadlock. A nested monitor lockout occurs like this:

```java
Thread 1 synchronizes on A
Thread 1 synchronizes on B (while synchronized on A)
Thread 1 decides to wait for a signal from another thread before continuing
Thread 1 calls B.wait() thereby releasing the lock on B, but not A.

Thread 2 needs to lock both A and B (in that sequence)
        to send Thread 1 the signal.
Thread 2 cannot lock A, since Thread 1 still holds the lock on A.
Thread 2 remain blocked indefinately waiting for Thread1
        to release the lock on A

Thread 1 remain blocked indefinately waiting for the signal from
        Thread 2, thereby
        never releasing the lock on A, that must be released to make
        it possible for Thread 2 to send the signal to Thread 1, etc.
```

This may sound like a pretty theoretical situation, but look at the naive Lock implemenation below:

```java
//lock implementation with nested monitor lockout problem

public class Lock{
  protected MonitorObject monitorObject = new MonitorObject();
  protected boolean isLocked = false;

  public void lock() throws InterruptedException{
    synchronized(this){
      while(isLocked){
        synchronized(this.monitorObject){
            this.monitorObject.wait();
        }
      }
      isLocked = true;
    }
  }

  public void unlock(){
    synchronized(this){
      this.isLocked = false;
      synchronized(this.monitorObject){
        this.monitorObject.notify();
      }
    }
  }
}
```

Notice how the lock() method first synchronizes on "this", then synchronizes on the monitorObject member. If isLocked is false there is no problem. The thread does not call monitorObject.wait(). If isLocked is true however, the thread calling lock() is parked waiting in the monitorObject.wait() call.

The problem with this is, that the call to monitorObject.wait() only releases the synchronization monitor on the monitorObject member, and not the synchronization monitor associated with "this". In other words, the thread that was just parked waiting is still holding the synchronization lock on "this".

When the thread that locked the Lock in the first place tries to unlock it by calling unlock() it will be blocked trying to enter the synchronized(this) block in the unlock() method. It will remain blocked until the thread waiting in lock() leaves the synchronized(this) block. But the thread waiting in the lock() method will not leave that block until the isLocked is set to false, and a monitorObject.notify() is executed, as it happens in unlock().

Put shortly, the thread waiting in lock() needs an unlock() call to execute successfully for it to exit lock() and the synchronized blocks inside it. But, no thread can actually execute unlock() until the thread waiting in lock() leaves the outer synchronized block.

This result is that any thread calling either lock() or unlock() will become blocked indefinately. This is called a nested monitor lockout.

## A More Realistic Example

You may claim that you would never implement a lock like the one shown earlier. That you would not call wait() and notify() on an internal monitor object, but rather on the This is probably true. But there are situations in which designs like the one above may arise. For instance, if you were to implement fairness in a Lock. When doing so you want each thread to call wait() on each their own queue object, so that you can notify the threads one at a time.

Look at this naive implementation of a fair lock:

```java
//Fair Lock implementation with nested monitor lockout problem

public class FairLock {
  private boolean           isLocked       = false;
  private Thread            lockingThread  = null;
  private List<QueueObject> waitingThreads =
            new ArrayList<QueueObject>();

  public void lock() throws InterruptedException{
    QueueObject queueObject = new QueueObject();

    synchronized(this){
      waitingThreads.add(queueObject);

      while(isLocked || waitingThreads.get(0) != queueObject){

        synchronized(queueObject){
          try{
            queueObject.wait();
          }catch(InterruptedException e){
            waitingThreads.remove(queueObject);
            throw e;
          }
        }
      }
      waitingThreads.remove(queueObject);
      isLocked = true;
      lockingThread = Thread.currentThread();
    }
  }

  public synchronized void unlock(){
    if(this.lockingThread != Thread.currentThread()){
      throw new IllegalMonitorStateException(
        "Calling thread has not locked this lock");
    }
    isLocked      = false;
    lockingThread = null;
    if(waitingThreads.size() > 0){
      QueueObject queueObject = waitingThreads.get(0);
      synchronized(queueObject){
        queueObject.notify();
      }
    }
  }
}
```

```java
public class QueueObject {}
```

At first glance this implementation may look fine, but notice how the lock() method calls queueObject.wait(); from inside two synchronized blocks. One synchronized on "this", and nested inside that, a block synchronized on the queueObject local variable. When a thread calls queueObject.wait()it releases the lock on the QueueObject instance, but not the lock associated with "this".

Notice too, that the unlock() method is declared synchronized which equals a synchronized(this) block. This means, that if a thread is waiting inside lock() the monitor object associated with "this" will be locked by the waiting thread. All threads calling unlock() will remain blocked indefinately, waiting for the waiting thread to release the lock on "this". But this will never happen, since this only happens if a thread succeeds in sending a signal to the waiting thread, and this can only be sent by executing the unlock() method.

And so, the FairLock implementation from above could lead to nested monitor lockout. A better implementation of a fair lock is described in the text Starvation and Fairness.

## Nested Monitor Lockout vs. Deadlock

The result of nested monitor lockout and deadlock are pretty much the same: The threads involved end up blocked forever waiting for each other.

The two situations are not equal though. As explained in the text on Deadlock a deadlock occurs when two threads obtain locks in different order. Thread 1 locks A, waits for B. Thread 2 has locked B, and now waits for A. As explained in the text on Deadlock Prevention deadlocks can be avoided by always locking the locks in the same order (Lock Ordering). However, a nested monitor lockout occurs exactly by two threads taking the locks in the same order. Thread 1 locks A and B, then releases B and waits for a signal from Thread 2. Thread 2 needs both A and B to send Thread 1 the signal. So, one thread is waiting for a signal, and another for a lock to be released.

The difference is summed up here:

```java
In deadlock, two threads are waiting for each other to release locks.

In nested monitor lockout, Thread 1 is holding a lock A, and waits
for a signal from Thread 2. Thread 2 needs the lock A to send the
signal to Thread 1.
```

The End.
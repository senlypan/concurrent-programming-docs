# Slipped Conditions

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/slipped-conditions.html  Update: 2022-02-24

## ⛔抱歉，本文暂无中文翻译
?> ❤️ 您也可以参与翻译，快来提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿参与吧~

## What is Slipped Conditions?

Slipped conditions means, that from the time a thread has checked a certain condition until it acts upon it, the condition has been changed by another thread so that it is errornous for the first thread to act. Here is a simple example:

```java
public class Lock {

    private boolean isLocked = true;

    public void lock(){
      synchronized(this){
        while(isLocked){
          try{
            this.wait();
          } catch(InterruptedException e){
            //do nothing, keep waiting
          }
        }
      }

      synchronized(this){
        isLocked = true;
      }
    }

    public synchronized void unlock(){
      isLocked = false;
      this.notify();
    }

}
```

Notice how the `lock()` method contains two synchronized blocks. The first block waits until `isLocked` is false. The second block sets isLocked to true, to lock the Lock instance for other threads.

Imagine that `isLocked` is false, and two threads call `lock()` at the same time. If the first thread entering the first synchronized block is preempted right after the first synchronized block, this thread will have checked isLocked and noted it to be false. If the second thread is now allowed to execute, and thus enter the first synchronized block, this thread too will see isLocked as false. Now both threads have read the condition as false. Then both threads will enter the second synchronized block, set isLocked to true, and continue.

This situation is an example of slipped conditions. Both threads test the condition, then exit the synchronized block, thereby allowing other threads to test the condition, before any of the two first threads change the conditions for subsequent threads. In other words, the condition has slipped from the time the condition was checked until the threads change it for subsequent threads.

To avoid slipped conditions the testing and setting of the conditions must be done atomically by the thread doing it, meaning that no other thread can check the condition in between the testing and setting of the condition by the first thread.

The solution in the example above is simple. Just move the line `isLocked = true`; up into the first synchronized block, right after the while loop. Here is how it looks:

```java
public class Lock {

    private boolean isLocked = true;

    public void lock(){
      synchronized(this){
        while(isLocked){
          try{
            this.wait();
          } catch(InterruptedException e){
            //do nothing, keep waiting
          }
        }
        isLocked = true;
      }
    }

    public synchronized void unlock(){
      isLocked = false;
      this.notify();
    }

}
```

Now the testing and setting of the isLocked condition is done atomically from inside the same synchronized block.

## A More Realistic Example

You may rightfully argue that you would never implement a Lock like the first implementation shown in this text, and thus claim slipped conditions to be a rather theoretical problem. But the first example was kept rather simple to better convey the notion of slipped conditions.

A more realistic example would be during the implementation of a fair lock, as discussed in the text on Starvation and Fairness. If we look at the naive implementation from the text Nested Monitor Lockout, and try to remove the nested monitor lock problem it, it is easy to arrive at an implementation that suffers from slipped conditions. First I'll show the example from the nested monitor lockout text:

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
      QueueObject queueObject = waitingThread.get(0);
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

Notice how the `synchronized(queueObject)` with its `queueObject.wait()` call is nested inside the `synchronized(this)` block, resulting in the nested monitor lockout problem. To avoid this problem the `synchronized(queueObject)` block must be moved outside the `synchronized(this)` block. Here is how that could look:

```java
//Fair Lock implementation with slipped conditions problem

public class FairLock {
  private boolean           isLocked       = false;
  private Thread            lockingThread  = null;
  private List<QueueObject> waitingThreads =
            new ArrayList<QueueObject>();

  public void lock() throws InterruptedException{
    QueueObject queueObject = new QueueObject();

    synchronized(this){
      waitingThreads.add(queueObject);
    }

    boolean mustWait = true;
    while(mustWait){

      synchronized(this){
        mustWait = isLocked || waitingThreads.get(0) != queueObject;
      }

      synchronized(queueObject){
        if(mustWait){
          try{
            queueObject.wait();
          }catch(InterruptedException e){
            waitingThreads.remove(queueObject);
            throw e;
          }
        }
      }
    }

    synchronized(this){
      waitingThreads.remove(queueObject);
      isLocked = true;
      lockingThread = Thread.currentThread();
    }
  }
}
```

Note: Only the `lock()` method is shown, since it is the only method I have changed.

Notice how the `lock()` method now contains 3 synchronized blocks.

The first `synchronized(this)` block checks the condition by setting `mustWait = isLocked` || `waitingThreads.get(0) != queueObject`.

The second synchronized(queueObject) block checks if the thread is to wait or not. Already at this time another thread may have unlocked the lock, but lets forget that for the time being. Let's assume that the lock was unlocked, so the thread exits the `synchronized(queueObject)` block right away.

The third `synchronized(this)` block is only executed if `mustWait = false`. This sets the condition `isLocked` back to true etc. and leaves the `lock()` method.

Imagine what will happen if two threads call `lock()` at the same time when the lock is unlocked. First thread 1 will check the `isLocked` conditition and see it false. Then thread 2 will do the same thing. Then neither of them will wait, and both will set the state `isLocked` to true. This is a prime example of slipped conditions.

Removing the Slipped Conditions Problem
To remove the slipped conditions problem from the example above, the content of the last `synchronized(this)` block must be moved up into the first block. The code will naturally have to be changed a little bit too, to adapt to this move. Here is how it looks:

```java
//Fair Lock implementation without nested monitor lockout problem,
//but with missed signals problem.

public class FairLock {
  private boolean           isLocked       = false;
  private Thread            lockingThread  = null;
  private List<QueueObject> waitingThreads =
            new ArrayList<QueueObject>();

  public void lock() throws InterruptedException{
    QueueObject queueObject = new QueueObject();

    synchronized(this){
      waitingThreads.add(queueObject);
    }

    boolean mustWait = true;
    while(mustWait){

        
        synchronized(this){
            mustWait = isLocked || waitingThreads.get(0) != queueObject;
            if(!mustWait){
                waitingThreads.remove(queueObject);
                isLocked = true;
                lockingThread = Thread.currentThread();
                return;
            }
        } 

      synchronized(queueObject){
        if(mustWait){
          try{
            queueObject.wait();
          }catch(InterruptedException e){
            waitingThreads.remove(queueObject);
            throw e;
          }
        }
      }
    }
  }
}
```

Notice how the local variable `mustWait` is tested and set within the same synchronized code block now. Also notice, that even if the mustWait local variable is also checked outside the `synchronized(this)` code block, in the while(mustWait) clause, the value of the mustWait variable is never changed outside the `synchronized(this)`. A thread that evaluates `mustWait` to false will atomically also set the internal conditions (`isLocked`) so that any other thread checking the condition will evaluate it to true.

The return; statement in the `synchronized(this)` block is not necessary. It is just a small optimization. If the thread must not wait (`mustWait == false`), then there is no reason to enter the `synchronized(queueObject)` block and execute the `if`mustWait)` clause.

The observant reader will notice that the above implementation of a fair lock still suffers from a missed signal problem. Imagine that the FairLock instance is locked when a thread calls `lock()`. After the first `synchronized(this)` block mustWait is true. Then imagine that the thread calling `lock()` is preempted, and the thread that locked the lock calls unlock(). If you look at the `unlock()` implementation shown earlier, you will notice that it calls `queueObject.notify()`. But, since the thread waiting in `lock()` has not yet called `queueObject.wait()`, the call to `queueObject.notify()` passes into oblivion. The signal is missed. When the thread calling `lock()` right after calls `queueObject.wait()` it will remain blocked until some other thread calls `unlock()`, which may never happen.

The missed signals problems is the reason that the FairLock implementation shown in the text Starvation and Fairness has turned the QueueObject class into a semaphore with two methods: `doWait()` and `doNotify()`. These methods store and react the signal internally in the QueueObject. That way the signal is not missed, even if `doNotify()` is called before `doWait()`.

The End.

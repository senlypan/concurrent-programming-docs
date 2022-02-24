# Reentrance Lockout

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/reentrance-lockout.html  Update: 2022-02-24

## ⛔抱歉，本文暂无中文翻译
?> ❤️ 您也可以参与翻译，快来提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿参与吧~

Reentrance lockout is a situation similar to deadlock and nested monitor lockout. Reentrance lockout is also covered in part in the texts on Locks and Read / Write Locks.

Reentrance lockout may occur if a thread reenters a Lock, ReadWriteLock or some other synchronizer that is not reentrant. Reentrant means that a thread that already holds a lock can retake it. Java's synchronized blocks are reentrant. Therefore the following code will work without problems:

```java
public class Reentrant{

  public synchronized outer(){
    inner();
  }

  public synchronized inner(){
    //do something
  }
}
```

Notice how both `outer()` and `inner()` are declared synchronized, which in Java is equivalent to a `synchronized(this)` block. If a thread calls `outer()` there is no problem calling `inner()` from inside `outer()`, since both methods (or blocks) are synchronized on the same monitor `object ("this")`. If a thread already holds the lock on a monitor object, it has access to all blocks synchronized on the same monitor object. This is called reentrance. The thread can reenter any block of code for which it already holds the lock.

The following Lock implementation is not reentrant:

```java
public class Lock{

  private boolean isLocked = false;

  public synchronized void lock()
  throws InterruptedException{
    while(isLocked){
      wait();
    }
    isLocked = true;
  }

  public synchronized void unlock(){
    isLocked = false;
    notify();
  }
}
```

If a thread calls `lock()` twice without calling `unlock()` in between, the second call to `lock()` will block. A reentrance lockout has occurred.

To avoid reentrance lockouts you have two options:

1. Avoid writing code that reenters locks
2. Use reentrant locks

Which of these options suit your project best depends on your concrete situation. Reentrant locks often don't perform as well as non-reentrant locks, and they are harder to implement, but this may not necessary be a problem in your case. Whether or not your code is easier to implement with or without lock reentrance must be determined case by case.

The End.
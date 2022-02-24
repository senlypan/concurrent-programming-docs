# Creating and Starting Java Threads

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/creating-and-starting-threads.html  Update: 2022-02-23

## ⛔抱歉，本文暂无中文翻译
?> ❤️ 您也可以参与翻译，快来提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿参与吧~

A Java Thread is like a virtual CPU that can execute your Java code - inside your Java application. when a Java application is started its main() method is executed by the main thread - a special thread that is created by the Java VM to run your application. From inside your application you can create and start more threads which can execute parts of your application code in parallel with the main thread.

Java threads are objects like any other Java objects. Threads are instances of class java.lang.Thread, or instances of subclasses of this class. In addition to being objects, java threads can also execute code. In this Java thread tutorial I will explain how to create and start threads.

## Java Threads Video Tutorial

In case you prefer video, here is a video version of this [Java Threads tutorial](https://www.youtube.com/watch?v=eQk5AWcTS8w).

!(02-Creating-and-Starting-Java-Threads#java-threads-video-screenshot.jpg)[http://tutorials.jenkov.com/images/java-concurrency/java-threads-video-screenshot.jpg]

## Creating and Starting Threads

Creating a thread in Java is done like this:

```java
    Thread thread = new Thread();
```

To start the Java thread you will call its start() method, like this:

```java
    thread.start();
```

This example doesn't specify any code for the thread to execute. Therfore the thread will stop again right away after it is started.

There are two ways to specify what code the thread should execute. The first is to create a subclass of Thread and override the `run()` method. The second method is to pass an object that implements `Runnable` (`java.lang.Runnable` to the `Thread` constructor. Both methods are covered below.

## Thread Subclass

The first way to specify what code a thread is to run, is to create a subclass of Thread and override the `run()` method. The `run()` method is what is executed by the thread after you call `start()`. Here is an example of creating a Java `Thread` subclass:

```java
  public class MyThread extends Thread {

    public void run(){
       System.out.println("MyThread running");
    }
  }
```

To create and start the above thread you can do like this:

```java
  MyThread myThread = new MyThread();
  myTread.start();
```

The `start()` call will return as soon as the thread is started. It will not wait until the `run()` method is done. The `run()` method will execute as if executed by a different CPU. When the `run()` method executes it will print out the text "MyThread running".

You can also create an anonymous subclass of `Thread` like this:

```java
  Thread thread = new Thread(){
    public void run(){
      System.out.println("Thread Running");
    }
  }

  thread.start();
```

This example will print out the text "Thread running" once the run() method is executed by the new thread.

## Runnable Interface Implementation

The second way to specify what code a thread should run is by creating a class that implements the `java.lang.Runnable` interface. A Java object that implements the `Runnable` interface can be executed by a Java `Thread`. How that is done is shown a bit later in this tutorial.

The `Runnable` interface is a standard [Java Interface](http://tutorials.jenkov.com/java/interfaces.html) that comes with the Java platform. The `Runnable` interface only has a single method `run()`. Here is basically how the `Runnable` interface looks:

```java
public interface Runnable() {

    public void run();

}
```

Whatever the thread is supposed to do when it executes must be included in the implementation of the `run()` method. There are three ways to implement the `Runnable` interface:

1. Create a Java class that implements the `Runnable` interface.
2. Create an anonymous class that implements the `Runnable` interface.
3. Create a Java Lambda that implements the `Runnable` interface.

All three options are explained in the following sections.

### Java Class Implements Runnable

The first way to implement the Java `Runnable` interface is by creating your own Java class that implements the `Runnable` interface. Here is an example of a custom Java class that implements the `Runnable` interface:

```java
  public class MyRunnable implements Runnable {

    public void run(){
       System.out.println("MyRunnable running");
    }
  }
```

All this `Runnable` implementation does is to print out the text `MyRunnable running`. After printing that text, the `run()` method exits, and the thread running the `run()` method will stop.

### Anonymous Implementation of Runnable

You can also create an anonymous implementation of `Runnable`. Here is an example of an anonymous Java class that implements the `Runnable` interface:

```java
Runnable myRunnable =
    new Runnable(){
        public void run(){
            System.out.println("Runnable running");
        }
    }
```

Apart from being an anononymous class, this example is quite similar to the example that used a custom class to implement the `Runnable` interface.

### Java Lambda Implementation of Runnable

The third way to implement the `Runnable` interface is by creating a [Java Lambda](http://tutorials.jenkov.com/java/lambda-expressions.html) implementation of the `Runnable` interface. This is possible because the `Runnable` interface only has a single unimplemented method, and is therefore practically (although possibly unintentionally) a [functional Java interface](http://tutorials.jenkov.com/java-functional-programming/functional-interfaces.html).

Here is an example of a Java lambda expression that implements the `Runnable` interface:

```java
Runnable runnable =
        () -> { System.out.println("Lambda Runnable running"); };
```

### Starting a Thread With a Runnable

To have the `run()` method executed by a thread, pass an instance of a class, anonymous class or lambda expression that implements the `Runnable` interface to a `Thread` in its constructor. Here is how that is done:

```java
Runnable runnable = new MyRunnable(); // or an anonymous class, or lambda...

Thread thread = new Thread(runnable);
thread.start();
```

When the thread is started it will call the `run()` method of the `MyRunnable` instance instead of executing it's own `run()` method. The above example would print out the text "MyRunnable running".

## Subclass or Runnable?

There are no rules about which of the two methods that is the best. Both methods works. Personally though, I prefer implementing `Runnable`, and handing an instance of the implementation to a Thread instance. When having the `Runnable`'s executed by a [thread pool](http://tutorials.jenkov.com/java-concurrency/creating-and-starting-threads.html) it is easy to queue up the  `Runnable` instances until a thread from the pool is idle. This is a little harder to do with `Thread` subclasses.

Sometimes you may have to implement `Runnable` as well as subclass `Thread`. For instance, if creating a subclass of `Thread` that can execute more than one `Runnable`. This is typically the case when implementing a thread pool.

## Common Pitfall: Calling run() Instead of start()

When creating and starting a thread a common mistake is to call the `run()` method of the Thread instead of `start()`, like this:

```java
  Thread newThread = new Thread(MyRunnable());
  newThread.run();  //should be start();
```

At first you may not notice anything because the Runnable's `run()` method is executed like you expected. However, it is NOT executed by the new thread you just created. Instead the `run()` method is executed by the thread that created the thread. In other words, the thread that executed the above two lines of code. To have the `run()` method of the MyRunnable instance called by the new created thread, `newThread`, you MUST call the `newThread.start()` method.

## Thread Names

When you create a Java thread you can give it a name. The name can help you distinguish different threads from each other. For instance, if multiple threads write to `System.out` it can be handy to see which thread wrote the text. Here is an example:

```java
   Thread thread = new Thread("New Thread") {
      public void run(){
        System.out.println("run by: " + getName());
      }
   };

   thread.start();
   System.out.println(thread.getName());
```

Notice the string "New Thread" passed as parameter to the `Thread` constructor. This string is the name of the thread. The name can be obtained via the Thread's `getName()` method. You can also pass a name to a `Thread` when using a `Runnable` implementation. Here is how that looks:

```java
   MyRunnable runnable = new MyRunnable();
   Thread thread = new Thread(runnable, "New Thread");

   thread.start();
   System.out.println(thread.getName());
```

Notice however, that since the `MyRunnable` class is not a subclass of `Thread`, it does not have access to the `getName()` method of the thread executing it.

## Thread.currentThread()

The `Thread.currentThread()` method returns a reference to the `Thread` instance executing `currentThread()` . This way you can get access to the Java `Thread` object representing the thread executing a given block of code. Here is an example of how to use `Thread.currentThread()` :

```java
Thread thread = Thread.currentThread();
```

Once you have a reference to the `Thread` object, you can call methods on it. For instance, you can get the name of the thread currently executing the code like this:

```java
   String threadName = Thread.currentThread().getName();
```

## Java Thread Example

Here is a small example. First it prints out the name of the thread executing the `main()` method. This thread is assigned by the JVM. Then it starts up 10 threads and give them all a number as name (`"" + i`). Each thread then prints its name out, and then stops executing.

```java
public class ThreadExample {

  public static void main(String[] args){
    System.out.println(Thread.currentThread().getName());
    for(int i=0; i<10; i++){
      new Thread("" + i){
        public void run(){
          System.out.println("Thread: " + getName() + " running");
        }
      }.start();
    }
  }
}
```

Note that even if the threads are started in sequence (1, 2, 3 etc.) they may not execute sequentially, meaning thread 1 may not be the first thread to write its name to `System.out`. This is because the threads are in principle executing in parallel and not sequentially. The JVM and/or operating system determines the order in which the threads are executed. This order does not have to be the same order in which they were started.

## Pause a Thread

A thread can pause itself by calling the static method `Thread.sleep()` . The `sleep()` takes a number of milliseconds as parameter. The `sleep()` method will attempt to sleep that number of milliseconds before resuming execution. The Thread `sleep()` is not 100% precise, but it is pretty good still. Here is an example of pausing a Java thread for 3 seconds (3.000 millliseconds) by calling the `Thread sleep()` method:

```java
try {
    Thread.sleep(10L * 1000L);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

The thread executing the Java code above, will sleep for approximately 10 seconds (10.000 milliseconds).

#Stop a Thread

Stopping a Java Thread requires some preparation of your thread implementation code. The Java `Thread` class contains a `stop()` method, but it is deprecated. The original `stop()` method would not provide any guarantees about in what state the thread was stopped. That means, that all Java objects the thread had access to during execution would be left in an unknown state. If other threads in your application also has access to the same objects, your application could fail unexpectedly and unpredictably.

Instead of calling the `stop()` method you will have to implement your thread code so it can be stopped. Here is an example of a class that implements `Runnable` which contains an extra method called `doStop()` which signals to the `Runnable` to stop. The `Runnable` will check this signal and stop when it is ready to do so.

```java
public class MyRunnable implements Runnable {

    private boolean doStop = false;

    public synchronized void doStop() {
        this.doStop = true;
    }

    private synchronized boolean keepRunning() {
        return this.doStop == false;
    }

    @Override
    public void run() {
        while(keepRunning()) {
            // keep doing what this thread should do.
            System.out.println("Running");

            try {
                Thread.sleep(3L * 1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }
}
```

This example first creates a `MyRunnable` instance, then passes that instance to a thread and starts the thread. Then the thread executing the `main()` method (the main thread) sleeps for 10 seconds, and then calls the `doStop()` method of the `MyRunnable` instance. This will cause the thread executing the `MyRunnable` method to stop, because the `keepRunning()` will return `false` after `doStop()` has been called.

Please keep in mind that if your `Runnable` implementation needs more than just the `run()` method (e.g. a `stop()` or `pause()` method too), then you can no longer create your `Runnable` implementation with a Java lambda expression. A Java lambda can only implement a single method. Instead you must use a custom class, or a custom interface that extends `Runnable` which has the extra methods, and which is implemented by an anonymous class.

The End.
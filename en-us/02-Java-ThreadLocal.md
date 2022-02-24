# Java ThreadLocal

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/threadlocal.html  Update: 2022-02-24

The Java ThreadLocal class enables you to create variables that can only be read and written by the same thread. Thus, even if two threads are executing the same code, and the code has a reference to the same ThreadLocal variable, the two threads cannot see each other's ThreadLocal variables. Thus, the Java ThreadLocal class provides a simple way to make code thread safe that would not otherwise be so.

## Java ThreadLocal Tutorial Video

If you prefer video, I have a video version of this Java ThreadLocal tutorial here:
[Java ThreadLocal Tutorial](https://www.youtube.com/watch?v=a_BoqsnVR2U&list=PLL8woMHwr36EDxjUoCzboZjedsnhLP1j4&index=9)

![02-Java-ThreadLocal.md#](http://tutorials.jenkov.com/images/java-concurrency/java-threadlocal-video-screenshot.jpg)

## Creating a ThreadLocal

You create a ThreadLocal instance just like you create any other Java object - via the new operator. Here is an example that shows how to create a ThreadLocal variable:

```java
private ThreadLocal threadLocal = new ThreadLocal();
```

This only needs to be done once per thread. Multiple threads can now get and set values inside this ThreadLocal, and each thread will only see the value it set itself.

## Set ThreadLocal Value

Once a ThreadLocal has been created you can set the value to be stored in it using its `set()` method.

```java
threadLocal.set("A thread local value");
```

## Get ThreadLocal Value

You read the value stored in a ThreadLocal using its get() method. Here is an example obtaining the value stored inside a Java ThreadLocal:

```java
String threadLocalValue = (String) threadLocal.get();
```

## Remove ThreadLocal Value

It is possible to remove a value set in a ThreadLocal variable. You remove a value by calling the ThreadLocal remove() method. Here is an example of removing the value set on a Java ThreadLocal:

```java
threadLocal.remove();
```

## Generic ThreadLocal

You can create a `ThreadLocal` with a generic type. Using a generic type only objects of the generic type can be set as value on the ThreadLocal. Additionally, you do not have to typecast the value returned by `get()`. Here is a generic `ThreadLocal` example:

```java
private ThreadLocal<String> myThreadLocal = new ThreadLocal<String>();
```

Now you can only store strings in the `ThreadLocal` instance. Additionally, you do not need to typecast the value obtained from the `ThreadLocal`:

```java
myThreadLocal.set("Hello ThreadLocal");

String threadLocalValue = myThreadLocal.get();
```

## Initial ThreadLocal Value

It is possible to set an initial value for a Java ThreadLocal which will get used the first time get() is called - before set() has been called with a new value. You have two options for specifying an initial value for a ThreadLocal:

- Create a ThreadLocal subclass that overrides the initialValue() method.
- Create a ThreadLocal with a Supplier interface implementation.

I will show you both options in the following sections.

### Override initialValue()

The first way to specify an initial value for a Java ThreadLocal variable is to create a subclass of ThreadLocal which overrides its initialValue() method. The easiest way to create a subclass of ThreadLocal is to simply create an anonymous subclass, right where you create the ThreadLocal variable. Here is an example of creating an anonymous subclass of ThreadLocal which overrides the initialValue() method:

```java
private ThreadLocal myThreadLocal = new ThreadLocal<String>() {
    @Override protected String initialValue() {
        return String.valueOf(System.currentTimeMillis());
    }
};  
```

Note, that different threads will still see different initial values. Each thread will create its own initial value. Only if you return the exact same object from the initialValue() method, will all threads see the same object. However, the whole point of using a ThreadLocal in the first place is to avoid the different threads seeing the same instance.

### Provide a Supplier Implementation

The second method for specifying an initial value for a Java ThreadLocal variable is to use its static factory method withInitial(Supplier) passing a Supplier interface implementation as parameter. This Supplier implementation supplies the initial value for the ThreadLocal. Here is an example of creating a ThreadLocal using its withInitial() static factory method, passing a simple Supplier implementation as parameter:

```java
ThreadLocal<String> threadLocal = ThreadLocal.withInitial(new Supplier<String>() {
    @Override
    public String get() {
        return String.valueOf(System.currentTimeMillis());
    }
});
```

Since Supplier is a functional interface, it an be implemented using a Java Lambda Expression. Here is how providing a Supplier implementation as a lambda expression to withInitial() looks:

```java
ThreadLocal threadLocal = ThreadLocal.withInitial(
        () -> { return String.valueOf(System.currentTimeMillis()); } );
```

As you can see, this is somewhat shorter than the previous example. But it can be made even a bit shorter yet, using the most dense syntax of lambda expressions:

```java
ThreadLocal threadLocal3 = ThreadLocal.withInitial(
        () -> String.valueOf(System.currentTimeMillis()) );
```

## Lazy Setting of ThreadLocal Value

In some situations you cannot use the standard ways of setting an initial value. For instance, perhaps you need some configuration information which is not available at the time you create the ThreadLocal variable. In that case you can set the initial value lazily. Here is an example of how setting an initial value lazily on a Java ThreadLocal:

```java
public class MyDateFormatter {

    private ThreadLocal<SimpleDateFormat> simpleDateFormatThreadLocal = new ThreadLocal<>();

    public String format(Date date) {
        SimpleDateFormat simpleDateFormat = getThreadLocalSimpleDateFormat();
        return simpleDateFormat.format(date);
    }
    
    
    private SimpleDateFormat getThreadLocalSimpleDateFormat() {
        SimpleDateFormat simpleDateFormat = simpleDateFormatThreadLocal.get();
        if(simpleDateFormat == null) {
            simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            simpleDateFormatThreadLocal.set(simpleDateFormat);
        }
        return simpleDateFormat;
    }
}
```
Notice how the format() method calls the getThreadLocalSimpleDateFormat() method to obtain a Java SimpleDatFormat instance. If a SimpleDateFormat instance has not been set in the ThreadLocal, a new SimpleDateFormat is created and set in the ThreadLocal variable. Once a thread has set its own SimpleDateFormat in the ThreadLocal variable, the same SimpleDateFormat object is used for that thread going forward. But only for that thread. Each thread creates its own SimpleDateFormat instance, as they cannot see each others instances set on the ThreadLocal variable.

The SimpleDateFormat class is not thread safe, so multiple threads cannot use it at the same time. To solve this problem, the MyDateFormatter class above creates a SimpleDateFormat per thread, so each thread calling the format() method will use its own SimpleDateFormat instance.

## Using a ThreadLocal with a Thread Pool or ExecutorService

If you plan to use a Java ThreadLocal from inside a task passed to a Java Thread Pool or a Java ExecutorService, keep in mind that you do not have any guarantees which thread will execute your task. However, if all you need is to make sure that each thread uses its own instance of some object, this is not a problem. Then you can use a Java ThreadLocal with a thread pool or ExecutorService just fine.

## Full ThreadLocal Example

Here is a fully runnable Java ThreadLocal example:

```java
public class ThreadLocalExample {

    public static void main(String[] args) {
        MyRunnable sharedRunnableInstance = new MyRunnable();

        Thread thread1 = new Thread(sharedRunnableInstance);
        Thread thread2 = new Thread(sharedRunnableInstance);

        thread1.start();
        thread2.start();

        thread1.join(); //wait for thread 1 to terminate
        thread2.join(); //wait for thread 2 to terminate
    }

}
```

```java
public class MyRunnable implements Runnable {

    private ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>();

    @Override
    public void run() {
        threadLocal.set( (int) (Math.random() * 100D) );

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
        }

        System.out.println(threadLocal.get());
    }
}
```

This example creates a single MyRunnable instance which is passed to two different threads. Both threads execute the run() method, and thus sets different values on the ThreadLocal instance. If the access to the set() call had been synchronized, and it had not been a ThreadLocal object, the second thread would have overridden the value set by the first thread.

However, since it is a ThreadLocal object then the two threads cannot see each other's values. Thus, they set and get different values.

## InheritableThreadLocal

The InheritableThreadLocal class is a subclass of ThreadLocal. Instead of each thread having its own value inside a ThreadLocal, the InheritableThreadLocal grants access to values to a thread and all child threads created by that thread. Here is a full Java InheritableThreadLocal example:

```java
public class InheritableThreadLocalBasicExample {

    public static void main(String[] args) {

        ThreadLocal<String> threadLocal = new ThreadLocal<>();
        InheritableThreadLocal<String> inheritableThreadLocal =
                new InheritableThreadLocal<>();

        Thread thread1 = new Thread(() -> {
            System.out.println("===== Thread 1 =====");
            threadLocal.set("Thread 1 - ThreadLocal");
            inheritableThreadLocal.set("Thread 1 - InheritableThreadLocal");

            System.out.println(threadLocal.get());
            System.out.println(inheritableThreadLocal.get());

            Thread childThread = new Thread( () -> {
                System.out.println("===== ChildThread =====");
                System.out.println(threadLocal.get());
                System.out.println(inheritableThreadLocal.get());
            });
            childThread.start();
        });

        thread1.start();

        Thread thread2 = new Thread(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("===== Thread2 =====");
            System.out.println(threadLocal.get());
            System.out.println(inheritableThreadLocal.get());
        });
        thread2.start();
    }
}
```

This example creates a normal Java ThreadLocal and a Java InheritableThreadLocal. Then the example creates one thread which sets the value of the ThreadLocal and InheritableThreadLocal - and then creates a child thread which accesses the values of the ThreadLocal and InheritableThreadLocal. Only the value of the InheritableThreadLocal is visible to the child thread.

Finally the example creates a third thread which also tries to access both the ThreadLocal and InheritableThreadLocal - but which does not see any of the values stored by the first thread.

The output printed from running this example would look like this:

```java
===== Thread 1 =====
Thread 1 - ThreadLocal
Thread 1 - InheritableThreadLocal
===== ChildThread =====
null
Thread 1 - InheritableThreadLocal
===== Thread2 =====
null
null
```

The End.
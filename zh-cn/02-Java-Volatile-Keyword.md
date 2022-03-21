# 关于Volatile

> 作者: 雅各布·詹科夫
>
> 原文: http://tutorials.jenkov.com/java-concurrency/volatile.html  最后更新: 2022-02-24

Java的`volatile`关键字用于将Java变量标记为“存储在主内存中”。更准确地说，每次对`volatile`变量的读取都将从计算机主内存中读取，而不是从CPU缓存中读取，并且每次对`volatile`变量的写入都将写入主内存，而不仅仅写在CPU缓存。

事实上，自从 Java5 之后，`volatile` 关键字就不仅仅被用来保证 `volatile` 变量读写主内存。我将在以下内容解释这一点。

## Java volatile 教程视频

如果你喜欢视频，我在这里有这个 `Java volatile` 教程的视频版本:
[Java volatile 教程视频](https://www.youtube.com/watch?v=nhYIEqt-jvY)

![02-Java-Volatile-Keyword.md#java-volatile-video-screenshot.jpg](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-video-screenshot.jpg)

## 变量可见性问题

Java的`volatile`关键字在多线程处理中保证了共享变量的“可见性”。这听起来可能有点抽象，所以让我详细说明。

在多线程应用程序中，如果多个线程对同一个无声明`volatile`关键词的变量进行操作，出于性能原因，每个线程可以在处理变量时讲变量从主内存复制到CPU缓存中。如果你的计算机拥有多CPU，则每个线程就可能在不同的CPU上运行。这就意味着，每个线程都可以将变量复制在不同CPU的CPU缓存上。这在此处进行了说明：

![02-Java-Volatile-Keyword.md#java-volatile-1.png](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-1.png)

对于无声明`volatile`关键词的变量而言，无法保证Java虚拟机（JVM）何时将数据从主内存读取到CPU缓存，或者将数据从CPU缓存写入主内存。这就可能会导致几个问题，我将在以下部分内容解释这些问题。

想象一个场景，多个线程访问一个共享对象，该对象包含一个声明如下的计数器（counter）变量：

```java
public class SharedObject {

    public int counter = 0;

}
```

假设只有线程1会增加计数器（counter）变量的值，但是线程1和线程2会不时的读取这个计数器变量。

如果计数器（counter）变量没有声明`volatile`关键词，则无法保证计数器变量的值何时从CPU缓存写回主内存。这就意味着，每个CPU缓存上的计数器变量值和主内存中的变量值可能不一致。这种情况如下所示：

![02-Java-Volatile-Keyword.md#java-volatile-2.png](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-2.png)

一个线程的写操作还没有写回主内存，其他线程（每个线程自己有本地缓存，即CPU缓存）看不到变量的最新值，这就是“可见性”问题。一个线程的更新对其他相差是不可见的。

## Java volatile 可见性保证

Java的`volatile`关键字就是为了解决变量的可见性问题。通过对计数器（counter）变量声明`volatile`关键字，所有线程对该变量的写入都会被立即同步到主内存中，并且，所有线程对该变量的读取都会直接从主内存读取。

以下是计数器（counter）变量声明了关键字`volatile`的用法：

Here is how the `volatile` declaration of the `counter` variable looks:

```java
public class SharedObject {

    public volatile int counter = 0;

}
```

因此，声明了`volatile`关键字的变量，保证了其他线程对该变量的写入可见性。

在以上给出的场景中，一个线程（T1）修改了计数器变量，而另一个线程（T2）读取计数器变量（但是没有进行修改），这种场景下如果给计数器（counter）变量声明`volatile`关键字，就能够保证计数器（counter）变量的写入对线程（T2）是可见的。

但是如果线程（T1）和线程（T2）都对计数器（counter）变量进行了修改，那么给计数器（counter）变量声明`volatile`关键字是无法保证可见性的，稍后讨论。

### volatile 全局可见性保证

实际上，Java的`volatile`关键字可见性保证超过了`volatile`变量本身的可见性，可见性保证如下：

- 如果线程A写入一个`volatile`变量，而线程B随后读取了同一个`volatile`变量，那么所有变量的可见性，在线程A写入`volatile`变量之前对线程A可见，在线程B读取`volatile`变量之后对线程B同样可见。

- 如果线程A读取一个`volatile`变量，那么读取`volatile`变量时，对线程A可见的所有变量也会从主内存中重新读取。

让我用一个代码示例来说明:

```java
public class MyClass {
    private int years;
    private int months
    private volatile int days;


    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

`udpate()`方法写入三个变量，其中只有变量days声明为`volatile`。

> `volatile`关键字声明的变量，被写入时会直接从本地线程缓存刷新到主内存。

`volatile`的全局可见性保证，指的是当一个值被写入`days`时，所有对当前写入线程可见的变量也都会被写入到主内存。意思就是当一个值被写入`days`变量时，`year`变量和`months`变量也会被写入到主内存。

在读`years`，`months`和`days`的值时，你可以这样做：

```java
public class MyClass {
    private int years;
    private int months
    private volatile int days;

    public int totalDays() {
        int total = this.days;
        total += months * 30;
        total += years * 365;
        return total;
    }

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

注意，`totalDays()`方法会首先读取`days`变量的值到total变量中，当程序读取`days`变量时，也会从主内存读取`month`变量和`years`变量的值。因此你可以通过以上的读取顺序，来保证读取到三个变量`days`,`months`和`years`最新的值。

## 指令重排序的挑战

为了提高性能，一般允许 JVM 和 CPU 在保证程序语义不变的情况下对程序中的指令进行重新排序。例如：

```java
int a = 1;
int b = 2;

a++;
b++;
```

这些指令可以重新排序为以下顺序，而不会丢失程序的语义含义：

```java
int a = 1;
a++;

int b = 2;
b++;
```

然而，当其中一个变量是`volatile`关键字声明的变量时，指令重排就会遇到一些挑战。让我们看看之前教程中的`MyClass`类示例：

```java
public class MyClass {
    private int years;
    private int months
    private volatile int days;


    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

一旦`update()`方法将一个值写入days变量，那么写入years变量和months变量的最新值也会被写入到主内存当中。但是，如果Java虚拟机对指令进行重排，例如这样：

```java
public void update(int years, int months, int days){
    this.days   = days;
    this.months = months;
    this.years  = years;
}
```

当修改`days`变量时，仍然会将`months`变量和`years`变量的值写入主内存，但是这个节点是发生在新值写入`months`变量和`years`变量之前。因此`months`变量和`years`变量的最新值不可能正确地对其他线程可见。这种重排指令会导致语义发生改变。

针对这个问题Java提供了一个解决方案，我们往下看。

## Java volatile Happens-Before 规则

为了解决指令重新排序的挑战，除了可见性保证之外，Java的`volatile`关键字还提供了happens-before规则。happens-before规则保证：

- 如果其他变量的读写操作原先就发生在`volatile`变量写操作之前，那么其他变量的读写指令不能被重排序到volatile变量的写指令之后;
    - 在`volatile`变量写入之前的其他变量读写，Happens-Before 于`volatile`变量的写入。

> 注意：例如在`volatile`变量写入之后的其他变量读写，仍然可能被重排到`volatile`变量写入之前。只不过不能反着来，允许后面的读写重排到前面，但不允许前面的读写重排到后面。

- 如果其他变量的读写操作原先就发生在`volatile`变量读操作之后，那么其他变量的读写指令不能被重排序到volatile变量的读指令之前; 
    
> 注意：例如在`volatile`变量读之前的其他变量读取，可能被重排到`volatile`变量的读之后。只不过不能反着来，允许前面的读取重排到后面，但不允许后面的读取重排到前面。

上述的happens-before规则，确保了`volatile`关键字的可见性保证会被强制要求。

## volatile is Not Always Enough

Even if the volatile keyword guarantees that all reads of a volatile variable are read directly from main memory, and all writes to a volatile variable are written directly to main memory, there are still situations where it is not enough to declare a variable volatile.

In the situation explained earlier where only Thread 1 writes to the shared counter variable, declaring the counter variable volatile is enough to make sure that Thread 2 always sees the latest written value.

In fact, multiple threads could even be writing to a shared volatile variable, and still have the correct value stored in main memory, if the new value written to the variable does not depend on its previous value. In other words, if a thread writing a value to the shared volatile variable does not first need to read its value to figure out its next value.

As soon as a thread needs to first read the value of a volatile variable, and based on that value generate a new value for the shared volatile variable, a volatile variable is no longer enough to guarantee correct visibility. The short time gap in between the reading of the volatile variable and the writing of its new value, creates an race condition where multiple threads might read the same value of the volatile variable, generate a new value for the variable, and when writing the value back to main memory - overwrite each other's values.

The situation where multiple threads are incrementing the same counter is exactly such a situation where a volatile variable is not enough. The following sections explain this case in more detail.

Imagine if Thread 1 reads a shared counter variable with the value 0 into its CPU cache, increment it to 1 and not write the changed value back into main memory. Thread 2 could then read the same counter variable from main memory where the value of the variable is still 0, into its own CPU cache. Thread 2 could then also increment the counter to 1, and also not write it back to main memory. This situation is illustrated in the diagram below:

![02-Java Volatile Keyword#java-volatile-3.png](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-3.png)

Thread 1 and Thread 2 are now practically out of sync. The real value of the shared counter variable should have been 2, but each of the threads has the value 1 for the variable in their CPU caches, and in main memory the value is still 0. It is a mess! Even if the threads eventually write their value for the shared counter variable back to main memory, the value will be wrong.

## When is volatile Enough?

As I have mentioned earlier, if two threads are both reading and writing to a shared variable, then using the volatile keyword for that is not enough. You need to use a synchronized in that case to guarantee that the reading and writing of the variable is atomic. Reading or writing a volatile variable does not block threads reading or writing. For this to happen you must use the synchronized keyword around critical sections.

As an alternative to a synchronized block you could also use one of the many atomic data types found in the java.util.concurrent package. For instance, the AtomicLong or AtomicReference or one of the others.

In case only one thread reads and writes the value of a volatile variable and other threads only read the variable, then the reading threads are guaranteed to see the latest value written to the volatile variable. Without making the variable volatile, this would not be guaranteed.

The volatile keyword is guaranteed to work on 32 bit and 64 variables.

## Performance Considerations of volatile

Reading and writing of volatile variables causes the variable to be read or written to main memory. Reading from and writing to main memory is more expensive than accessing the CPU cache. Accessing volatile variables also prevent instruction reordering which is a normal performance enhancement technique. Thus, you should only use volatile variables when you really need to enforce visibility of variables.

（本篇完）

?> ✨ 译文来源：[潘深练](http://www.panshenlian.com) 如您有更好的翻译版本，欢迎 ❤️ 提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿哦~

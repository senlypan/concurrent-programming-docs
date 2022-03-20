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

想象一个场景，多个线程访问一个共享对象，该对象包含一个申明如下的计数器（counter）变量：

```java
public class SharedObject {

    public int counter = 0;

}
```

再想象以下，仅有线程1会增加计数器（counter）变量的值，不过线程1和线程2会不时的读取这个计数器变量。

如果计数器（counter）变量没有申明`volatile`关键词，则无法保证计数器变量的值何时从CPU缓存写回主内存。这就意味着，每个CPU缓存上的计数器变量值和主内存中的变量值可能不一致。这种情况如下所示：

![02-Java-Volatile-Keyword.md#java-volatile-2.png](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-2.png)

一个线程的写操作还没有写回主内存，其他线程（每个线程自己有本地缓存，即CPU缓存）看不到变量的最新值，这就是“可见性”问题。一个线程的更新对其他相差是不可见的。

## Java volatile 可见性保证

Java的`volatile`关键字就是为了解决变量的可见性问题。通过对计数器（counter）变量申明`volatile`关键字，所有线程对该变量的写入都会被立即同步到主内存中，并且，所有线程对该变量的读取都会直接从主内存读取。

以下是计数器（counter）变量申明了关键字`volatile`的用法：

Here is how the `volatile` declaration of the `counter` variable looks:

```java
public class SharedObject {

    public volatile int counter = 0;

}
```

因此，申明了`volatile`关键字的变量，保证了其他线程对该变量的写入可见性。

在以上给出的场景中，一个线程（T1）修改了计数器变量，而另一个线程（T2）读取计数器变量（但是没有进行修改），这种场景下如果给计数器（counter）变量声明`volatile`关键字，就能够保证计数器（counter）变量的写入对线程（T2）是可见的。

但是如果线程（T1）和线程（T2）都对计数器（counter）变量进行了修改，那么给计数器（counter）变量声明`volatile`关键字是无法保证可见性的，稍后讨论。

### volatile 全局可见性保证

Actually, the visibility guarantee of Java volatile goes beyond the volatile variable itself. The visibility guarantee is as follows:

- If Thread A writes to a volatile variable and Thread B subsequently reads the same volatile variable, then all variables visible to Thread A before writing the volatile variable, will also be visible to Thread B after it has read the volatile variable.

- If Thread A reads a volatile variable, then all all variables visible to Thread A when reading the volatile variable will also be re-read from main memory.

Let me illustrate that with a code example:

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

The `udpate()` method writes three variables, of which only days is volatile.

The full `volatile` visibility guarantee means, that when a value is written to `days`, then all variables visible to the thread are also written to main memory. That means, that when a value is written to `days`, the values of `years` and months are also written to main memory.

When reading the values of `years`, `months` and `days` you could do it like this:

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

Notice the totalDays() method starts by reading the value of days into the total variable. When reading the value of days, the values of months and years are also read into main memory. Therefore you are guaranteed to see the latest values of days, months and years with the above read sequence.

## Instruction Reordering Challenges

The Java VM and the CPU are allowed to reorder instructions in the program for performance reasons, as long as the semantic meaning of the instructions remain the same. For instance, look at the following instructions:

```java
int a = 1;
int b = 2;

a++;
b++;
```

These instructions could be reordered to the following sequence without losing the semantic meaning of the program:

```java
int a = 1;
a++;

int b = 2;
b++;
```

However, instruction reordering present a challenge when one of the variables is a volatile variable. Let us look at the MyClass class from the example earlier in this Java volatile tutorial:

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

Once the `update()` method writes a value to days, the newly written values to years and months are also written to main memory. But, what if the Java VM reordered the instructions, like this:

```java
public void update(int years, int months, int days){
    this.days   = days;
    this.months = months;
    this.years  = years;
}
```

The values of months and years are still written to main memory when the days variable is modified, but this time it happens before the new values have been written to months and years. The new values are thus not properly made visible to other threads. The semantic meaning of the reordered instructions has changed.

Java has a solution for this problem, as we will see in the next section.

## The Java volatile Happens-Before Guarantee

To address the instruction reordering challenge, the Java volatile keyword gives a "happens-before" guarantee, in addition to the visibility guarantee. The happens-before guarantee guarantees that:

- Reads from and writes to other variables cannot be reordered to occur after a write to a volatile variable, if the reads / writes originally occurred before the write to the volatile variable.
The reads / writes before a write to a volatile variable are guaranteed to "happen before" the write to the volatile variable. Notice that it is still possible for e.g. reads / writes of other variables located after a write to a volatile to be reordered to occur before that write to the volatile. Just not the other way around. From after to before is allowed, but from before to after is not allowed.
- Reads from and writes to other variables cannot be reordered to occur before a read of a volatile variable, if the reads / writes originally occurred after the read of the volatile variable. Notice that it is possible for reads of other variables that occur before the read of a volatile variable can be reordered to occur after the read of the volatile. Just not the other way around. From before to after is allowed, but from after to before is not allowed.

The above happens-before guarantee assures that the visibility guarantee of the volatile keyword are being enforced.

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

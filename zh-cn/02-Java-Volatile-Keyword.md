# 关于Volatile

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.concurrent.02-java-volatile-keyword&left_color=blue&right_color=red)

> 作者: 雅各布·詹科夫
>
> 原文: http://tutorials.jenkov.com/java-concurrency/volatile.html  最后更新: 2022-02-24

Java的`volatile`关键字用于将Java变量标记为“存储在主内存中”。更准确地说，每次对`volatile`变量的读取都将从计算机主内存中读取，而不是从CPU缓存中读取，并且每次对`volatile`变量的写入都将写入主内存，而不仅仅写在CPU缓存。

事实上，自从 Java5 开始，`volatile` 关键字就不仅仅被用来保证 `volatile` 变量读写主内存。我将在以下内容解释这一点。

## Java volatile 教程视频

如果你喜欢视频，我在这里有这个 `Java volatile` 教程的视频版本:
[Java volatile 教程视频](https://www.youtube.com/watch?v=nhYIEqt-jvY)

![02-Java-Volatile-Keyword.md#java-volatile-video-screenshot.jpg](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-video-screenshot.jpg)

## 变量可见性问题

Java的`volatile`关键字在多线程处理中保证了共享变量的“可见性”。这听起来可能有点抽象，所以让我详细说明。

在多线程应用程序中，如果多个线程对同一个无声明`volatile`关键词的变量进行操作，出于性能原因，每个线程可以在处理变量时将变量从主内存复制到CPU缓存中。如果你的计算机拥有多CPU，则每个线程可能在不同的CPU上运行。这就意味着，每个线程都可以将变量复制在不同CPU的CPU缓存上。这在此处进行了说明：

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

一个线程的写操作还没有写回主内存（每个线程都有本地缓存，即CPU缓存，一般写入成功会从cpu缓存刷新至主内存），其他线程看不到变量的最新值，这就是“可见性”问题，即一个线程的更新对其他线程是不可见的。

## Java volatile 可见性保证

Java的`volatile`关键字就是为了解决变量的可见性问题。通过对计数器（counter）变量声明`volatile`关键字，所有线程对该变量的写入都会被立即同步到主内存中，并且，所有线程对该变量的读取都会直接从主内存读取。

以下是计数器（counter）变量声明了关键字`volatile`的用法：

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

为了解决指令重新排序的挑战，除了可见性保证之外，Java的`volatile`关键字还提供了Happens-Before规则。Happens-Before规则保证：

- 如果其他变量的读写操作原先就发生在`volatile`变量的写操作之前，那么其他变量的读写指令不能被重排序到volatile变量的写指令之后;
    - 在`volatile`变量写入之前，发生的其他变量的读写，Happens-Before 于`volatile`变量的写入。

> 注意：例如在`volatile`变量写入之后的其他变量读写，仍然可能被重排到`volatile`变量写入之前。只不过不能反着来，允许后面的读写重排到前面，但不允许前面的读写重排到后面。

- 如果其他变量的读写操作原先就发生在`volatile`变量读操作之后，那么其他变量的读写指令不能被重排序到volatile变量的读指令之前; 

> 注意：例如在`volatile`变量读之前的其他变量读取，可能被重排到`volatile`变量的读之后。只不过不能反着来，允许前面的读取重排到后面，但不允许后面的读取重排到前面。

上述的Happens-Before规则，确保了`volatile`关键字的可见性保证会被强制要求。

## 仅声明 volatile 不足以保证线程安全

即使`volatile`关键字保证直接从主内存读取`volatile`变量，并且所有对`volatile`变量的写入都直接写入主内存，在某些情况下仅仅声明变量`volatile`是不足以保证线程安全的。

在前面解释的情况中，只有线程1写入共享计数器变量，声明计数器变量volatile足以确保线程2始终看到最新的写入值。

事实上，如果写入变量的新值不需要依赖之前的值，那多个线程可以同时对一个`volatile`共享变量进行写入操作，并且在主内存中仍然存储正确的值。换而言之，如果一个线程仅对一个`volatile`共享变量进行写入操作，那并不需要先读取出这个变量的值，再通过计算得到下一个值。

一旦线程需要首先读取出`volatile`变量的值，再基于该值为`volatile`共享变量生成新值，那`volatile`变量就不再足以保证正确的可见性。在读取`volatile`变量和写入新值之间的短暂时间会产生资源竞争，存在多个线程同时来读取`volatile`变量并得到相同的值，且都为变量赋予新值，然后将值都写回主内存中，从而会覆盖掉彼此的值。

多个线程递增同个计数器（counter）变量的情况，导致`volatile`变量不够保证线程安全性。 以下部分更详细地解释了这种情况:

想象一下，如果线程1将值为0的共享计数器（counter）变量读入其CPU高速缓存，则将其递增为1并且还未将更改的值写回主内存。 同时间线程2也可以从主内存中读取到相同的计数器变量，其中变量的值仍为0，存进其自己的CPU高速缓存。 然后，线程2也可以将计数器（counter）递增到1，也还未将其写回主内存。 这种情况如下图所示：

![02-Java Volatile Keyword#java-volatile-3.png](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-3.png)

线程1和线程2现在几乎不同步。共享计数器（counter）变量的实际值应该是2，但每个线程在其CPU缓存中的变量值为1，在主内存中该值仍然为0。真是一团糟！即使线程最终将其共享计数器变量的值写回主内存，该值也将是错误的。

## volatile 何时是线程安全的

正如我前面提到的，如果两个线程都在读取和写入共享变量，那么使用`volatile`关键字是不足以保证线程安全的。一般这种情况下，您需要使用`synchronized`来保证变量的读取和写入是原子性的。读取或写入`volatile`变量不会阻塞其他线程读取或写入。为此，您必须在关键部分周围使用`synchronized`关键字。

作为`synchronized`块的替代方案，您可以选择使用`java.util.concurrent`并发包中的原子数据类型。 例如，`AtomicLong`或`AtomicReference`或其它之一。

如果只有一个线程读取和写入`volatile`变量的值，而其他线程只读取变量，那么读取线程将保证看到写入`volatile`变量的最新值。 如果不使变量变为`volatile`，则无法保证。

`volatile`关键字保证适用于32位和64位。

## volatile 的性能注意事项

读写`volatile`变量都会直接从主内存读写，比从CPU缓存读写要花更多的开销，但访问`volatile`变量可以阻止指令重排，这是一项正常的性能增强技术。因此，除非确实需要强制实施变量的可见性，否则其他情况减少使用`volatile`变量。

（本篇完）

?> ✨ 译文来源：[潘深练](https://www.panshenlian.com) 如您有更好的翻译版本，欢迎 ❤️ 提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿哦~

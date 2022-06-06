# CAS

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.concurrent.04-compare-and-swap&left_color=blue&right_color=red)

> 作者: 雅各布·詹科夫
>
> 原文: http://tutorials.jenkov.com/java-concurrency/compare-and-swap.html  最后更新: 2022-02-24

`CAS` (compare and swap) 是并发算法设计时使用的一种技术。基本上，`CAS`是将变量的值与期望值进行比较，如果值相等，则将变量的值交换设置为新值。`CAS`可能听起来有点复杂，但一旦你理解它实际上相当简单，所以让我进一步详细说明这个主题。

顺便说一句，compare and swap 有时是 `CAS` 的缩写，所以如果你看到一些关于并发的文章或视频提到 `CAS`，它很有可能是指 compare and swap（比较并交换）操作。

## CAS教程视频

如果您喜欢视频，我在这里有这个`CAS`的视频教程版本：
[CAS视频教程](https://www.youtube.com/watch?v=ufWVK7CHOAk&list=PLL8woMHwr36EDxjUoCzboZjedsnhLP1j4&index=18)

![04-Compare-and-Swap.md#compare-and-swap-video-screenshot.png](http://tutorials.jenkov.com/images/java-concurrency/compare-and-swap-video-screenshot.png)

## CAS的使用场景（Check Then Act）

并发算法中常见的模式是先检查后执行（`check then act`）模式。当代码首先检查变量的值然后根据该值进行操作时，就会出现先检查后执行（`check then act`）模式。这是一个简单的例子：

```java
public class ProblematicLock {

    private volatile boolean locked = false;

    public void lock() {

        while(this.locked) {
            // 忙等待 - 直到 this.locked == false
        }

        this.locked = true;
    }

    public void unlock() {
        this.locked = false;
    }

}
```

此代码不是多线程锁的 `100%` 正确实现。这就是我给它命名的原因 `ProblematicLock` (问题锁) 。然而，我创建了这个错误的实现来说明如何通过`CAS`功能来解决它​​的问题。

该`lock()`方法首先检查成员变量是否`locked`等于`false`。这是在`while-loop`内部完成的。如果`locked`变量是`false`，则该`lock()`方法离开`while`循环并设置`locked`为`true`。换句话说，该 `lock()`方法首先检查变量的值`locked`，然后根据该检查进行操作。先检查，再执行。

如果多个线程几乎同时刻访问同一个 `ProblematicLock` 实例，那以上的 `lock()` 方法将会有一些问题，例如：

如果线程 A 检查`locked`的值为 `false`（预期值），它将退出 `while-loop` 循环执行后续的逻辑。如果此时有个线程B在线程A将`locked`值设置为 `true` 之前也检查了 `locked` 的值，那么线程B也将退出 `while-loop` 循环执行后续的逻辑。这是一个典型的资源竞争问题。

## 先检查后执行（Check Then Act）必须是原子性的

为了在多线程应用程序中正常工作（以避免资源竞争），先检查后执行（`Check Then Act`）必须是原子性的。原子性的意思是检查和执行动作都作为原子（不可分割的）代码块执行。任何开始执行该块的线程都将完成该块的执行，而不受其他线程的干扰。不允许其他线程在同一时刻执行相同原子块。

使`Java`代码块具有原子性的一种简单方法是使用`Java`的`synchronized`关键字对其进行标记。可以参阅[ 关于synchronized](../zh-cn/02-Java-Synchronized-Blocks.md) 的内容。这是`ProblematicLock`之前使用`synchronized`关键字将`lock()`方法转换为原子代码块的方法：

```java
public class MyLock {

    private volatile boolean locked = false;

    public synchronized void lock() {

        while(this.locked) {
            // 忙等待 - 直到 this.locked == false
        }

        this.locked = true;
    }

    public void unlock() {
        this.locked = false;
    }

}
```

现在方法`lock()`已申明同步，因此同一实例的`lock()`方法在同一时刻只允许被一个线程访问执行。相当于 `lock()` 方法是原子性的。

## 阻塞线程的代价很大

当两个线程试图同时进入`Java`中的一个同步块时，其中一个线程将被阻塞，而另一个线程将被允许进入同步块。当进入同步块的线程再次退出该块时，等待中的线程才会被允许进入该块。

如果线程被允许访问执行，那么进入一段同步代码块的代价并不大。但是如果因为已有一个线程在同步块中执行导致另一个线程被迫等阻塞，那么这个阻塞线程的代价就很大。

此外，当同步块再次空闲时，**您无法准确地确定何时能解除阻塞的线程**。这通常取决于`操作系统`或`执行平台`来 **协调** 阻塞线程的 **阻塞解除**。当然，在阻塞线程被解除阻塞并允许进入之前不会花费几秒钟或几分钟，但是可能会浪费一些时间用于阻塞线程，因为它本来可以访问共享数据结构的。这在此处进行了说明：

![04-Compare-and-Swap.md#compare-and-swap-1.png](http://tutorials.jenkov.com/images/java-concurrency/compare-and-swap-1.png)

## 硬件提供的原子性CAS操作

现代 `CPU` 内置了对`CAS`的原子性操作的支持。在某些情况下，可以使用`CAS`操作来替代同步块或其他阻塞数据结构。`CPU` 保证一次只有一个线程可以执行`CAS`操作，即使跨 `CPU` 内核也是如此。稍后在代码中有示例。

当使用硬件或 `CPU` 提供的`CAS`功能而不是操作系统或执行平台提供的 `synchronized`、`lock`、`mutex`（互斥锁） 等时，操作系统或执行平台不需要处理线程的阻塞和解除阻塞。这使得使用`CAS`的线程等待执行操作的时间更短，并且拥有更少的拥塞和更高的吞吐量。如下图所示：

![04-Compare-and-Swap.md#compare-and-swap-2.png](http://tutorials.jenkov.com/images/java-concurrency/compare-and-swap-2.png)

如您所见，试图进入共享数据结构的线程永远不会被完全阻塞。它不断尝试执行`CAS`操作，直到成功，并被允许访问共享数据结构。这样线程可以进入共享数据结构之前的延迟被最小化。

当然，如果线程在重复执行`CAS`的过程中等待很长时间，可能会浪费大量的`CPU`周期，而这些`CPU`周期本来可以用在其他任务（其他线程）上。但在许多情况下，情况并非如此。这取决于共享数据结构被另一个线程使用多长时间。实际上，共享数据结构的使用时间不长，因此上述情况不应该经常发生。但同样这取决于具体情况、代码、数据结构、尝试访问数据结构的线程数、系统负载等。相反，阻塞的线程根本不使用`CPU`。

## Java中的CAS

从 `Java 5` 开始，您可以通过`java.util.concurrent.atomic`包中的一些新的原子类访问 `CPU` 级别的`CAS`方法。这些类有：

- [AtomicBoolean](http://tutorials.jenkov.com/java-util-concurrent/atomicboolean.html)
- [AtomicInteger](http://tutorials.jenkov.com/java-util-concurrent/atomicinteger.html)
- [AtomicLong](http://tutorials.jenkov.com/java-util-concurrent/atomiclong.html)
- [AtomicReference](http://tutorials.jenkov.com/java-util-concurrent/atomicreference.html)
- [AtomicStampedReference](http://tutorials.jenkov.com/java-util-concurrent/atomicstampedreference.html)
- [AtomicIntegerArray](http://tutorials.jenkov.com/java-util-concurrent/atomicintegerarray.html)
- [AtomicLongArray](http://tutorials.jenkov.com/java-util-concurrent/atomiclongarray.html)
- [AtomicReferenceArray](http://tutorials.jenkov.com/java-util-concurrent/atomicreferencearray.html)

使用 `Java 5+` 附带的 `CAS` 功能而不是自己实现的优势在于，`Java 5+` 中内置的 `CAS` 功能允许您的应用程序利用 `CPU` 的底层能力执行`CAS`操作。这使您的`CAS`实现代码更快。

## CAS的保障性

`CAS`功能可用于保护临界区（`Critical Section`），从而防止多个线程同时执行临界区。

?> **critical section** 是每个线程中访问临界资源的那段代码，不论是硬件临界资源，还是软件临界资源，多个线程必须互斥地对它进行访问。每个线程中访问临界资源的那段代码称为临界区（`Critical Section`）。每个线程中访问临界资源的那段程序称为临界区（`Critical Section`）（临界资源是一次仅允许一个线程使用的**共享资源**）。每次只准许一个线程进入临界区，进入后不允许其他线程进入。

下面的一个示例，展示了如何使用`AtomicBoolean`类的`CAS`功能来实现前面显示的`lock()`方法并因此起到保障作用（一次只有一个线程可以退出该`lock()`方法）。

```java
public class CompareAndSwapLock {

    private AtomicBoolean locked = new AtomicBoolean(false);

    public void unlock() {
        this.locked.set(false);
    }

    public void lock() {
        while(!this.locked.compareAndSet(false, true)) {
            // busy wait - until compareAndSet() succeeds
        }
    }
}
```

注意这个`locked`变量不再是一个布尔类型而是`AtomicBoolean`类型，此类有一个`compareAndSet()`方法，会把实例的值（变量locked）与第一个参数（`false`）进行比较，如果比较结果相同（即locked的值等于第一个参数false），那么会将实例的值 `locked` 与期望值`true`交换（即把locked变量设置为true，表示锁住了）。如果交换成功则`compareAndSet()`方法会返回 `true`，如果没有交换成功则返回 `false`。

在上面的例子中，`compareAndSet()`方法调用比较了`locked`变量值与`false`值，如果`locked`变量值的结果值就是`false`，那么就是设置`locked`值为`true`。

由于一次只能允许一个线程执行该`compareAndSet()`方法，因此只有一个线程能够看到AtomicBoolean实例值为 `false`，从而将其交换为`true`。因此，每次只有一个线程能够退出`while-loop`（while循环），通过调用 `unlock()` 方法设置 `locked` 为 `false` 使得每次只有一个线程的 `CompareAndSwapLock` 是解锁状态的。 

## CAS实现乐观锁

也可以使用`CAS`功能作为乐观锁机制。乐观锁机制允许多个线程同时进入临界区，但只允许其中一个线程在临界区结束时提交其工作。

下面是一个使用乐观锁策略的并发计数器类示例：

```java
public class OptimisticLockCounter{

    private AtomicLong count = new AtomicLong();


    public void inc() {

        boolean incSuccessful = false;
        while(!incSuccessful) {
            long value = this.count.get();
            long newValue = value + 1;

            incSuccessful = this.count.compareAndSet(value, newValue);
        }
    }

    public long getCount() {
        return this.count.get();
    }
}
```

请注意 `inc()` 方法是如何从 `AtomicLong`实例变量`count`中获取现有计数值的。然后根据旧值计算出新值。最后，`inc()` 方法尝试通过调用`AtomicLong`实例的`compareAndSet()`方法来设置新值。

如果`AtomicLong`实例值`count`在比较时仍然拥有与上次获取时（`long value = this.count.get()`）的值相同，那么`compareAndSet()`会执行成功。但是假如有另一个线程在同一时刻已经调用增加了`AtomicLong`实例值（指有一个线程在之前已经调用成功`compareAndSet()`方法了，一般认为是**资源竞争**），则`compareAndSet()`调用将失败，因为预期值`value`不再等于存储在中的值`AtomicLong`（原值已经被前一个线程更改过）。在这种情况下，`inc()`方法将在 `while-loop`（while循环）中进行另外一次迭代并尝试再次增加`AtomicLong`值。

（本篇完）

?> ✨ 译文来源：[潘深练](https://www.panshenlian.com) 如您有更好的翻译版本，欢迎 ❤️ 提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿哦~
# Java Happens Before Guarantee

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/java-happens-before-guarantee.html  Update: 2022-02-23

## ⛔抱歉，本文暂无中文翻译，持续更新中
?> ❤️ 您也可以参与翻译，快来提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿参与吧~

The Java happens before guarantee is a set of rules that govern how the Java VM and CPU is allowed to reorder instructions for performance gains. The happens before guarantee makes it possible for threads to rely on when a variable value is synchronized to or from main memory, and which other variables have been synchronized at the same time. The Java happens before guarantee are centered around access to volatile variables and variables accessed from within synchronized blocks.

This Java happens before guarantee tutorial will mention the happens before guarantees provided by the Java volatile and Java synchronized declarations, but I will not explain everything about these declarations in this tutorial. These terms are covered in more detail here:

[Java volatile tutorial](http://tutorials.jenkov.com/java-concurrency/volatile.html)
[Java synchronized tutorial](http://tutorials.jenkov.com/java-concurrency/synchronized.html).

## Java Happens Before Guarantee Tutorial Video

If you prefer video, I have created a video version of this tutorial:
[Java Happens Before Guarantee](https://www.youtube.com/watch?v=oY14UyP61F8)

![02-Java Happens Before Guarantee#java-happens-before-guarantee-video-screenshot.jpg](http://tutorials.jenkov.com/images/java-concurrency/java-happens-before-guarantee-video-screenshot.jpg)

## Instruction Reordering

Modern CPUs have the ability to execute instructions in parallel if the instructions do not depend on each other. For instance, the following two instructions do not depend on each other, and can therefore be executed in parallel:

```java
a = b + c

d = e + f
```

However, the following two instructions cannot easily be executed in parallel, because the second instruction depends on the result of the first instruction:

```java
a = b + c
d = a + e
```

Imagine the two instructions above were part of a larger set of instructions, like the following:

```java
a = b + c
d = a + e

l = m + n
y = x + z
```

The instructions could be reordered like below. Then the CPU can execute at least the first 3 instructions in parallel, and as soon as the first instructions is finished, it can start executing the 4th instruction.  

```java
a = b + c

l = m + n
y = x + z

d = a + e
```

As you can see, reordering instructions can increase parallel execution of instructions in the CPU. Increased parallelization means increased performance.

Instruction reordering is allowed for the Java VM and the CPU as long as the semantics of the program do not change. The end result has to be the same as if the instructions were executed in the exact order they are listed in the source code.

## Instruction Reordering Problems in Multi CPU Computers

Instruction reordering poses some challenges in a multithreaded, multi CPU system. I will try to illustrate these problems through a code example. Keep in mind that the example is constructed specifically to illustrate these problems. Thus, the code example is not a recommendation in any way!

Imagine two threads that collaborate to draw frames on the screen as fast as they can. One frame generates the frame, and the other thread draws the frame on the screen.

The two threads need to exchange the frames via some communication mechanism. In the following code example I have created an example of such a communication mechanism - a Java class called FrameExchanger.

The frame producing thread produces frames as fast as it can. The frame drawing thread will draw those frames as fast as it can.

Sometimes the producer thread might produce 2 frames before the drawing thread has time to draw them. In that case, only the latest frame should be drawn. We don't want the drawing thread to fall behind the producing thread. In case the producer thread has a new frame ready before the previous frame has been drawn, the previous frame is simply overwritten with the new frame. In other words, the previous frame is "dropped".

Sometimes the drawing thread might draw a frame and be ready to draw a new frame before the producing thread has produced a new frame. In that case we want the drawing frame to wait for the new frame. There is no reason to waste CPU and GPU resources redraw the exact same frame that was just drawn! The screen won't change from that, and the user won't see anything new from that.

The FrameExchanger counts the number of frames stored, and the number of frames taken, so we can get a feeling for how many frames were dropped.

Below is the code for the FrameExchanger. Note: The Frame class definition is left out. It is not important how this class looks in order to understand how the FrameExchanger works. The producing thread will call ``storeFrame()` continuously, and the drawing thread will call `takeFrame()` continuously.

```java
public class FrameExchanger  {

    private long framesStoredCount = 0:
    private long framesTakenCount  = 0;

    private boolean hasNewFrame = false;

    private Frame frame = null;

        // called by Frame producing thread
    public void storeFrame(Frame frame) {
        this.frame = frame;
        this.framesStoredCount++;
        this.hasNewFrame = true;
    }

        // called by Frame drawing thread
    public Frame takeFrame() {
        while( !hasNewFrame) {
            //busy wait until new frame arrives
        }

        Frame newFrame = this.frame;
        this.framesTakenCount++;
        this.hasNewFrame = false;
        return newFrame;
    }

}
```

Notice how the three instructions inside the `storeFrame()` method seem like they do not depend on each other. That means, that to the Java VM and the CPU it looks like it would be okay to reorder the instructions, in case the Java VM or the CPU determines that would advantageous. However, imagine what would happen if the instructions were reordered, like this:

```java
    public void storeFrame(Frame frame) {
        this.hasNewFrame = true;
        this.framesStoredCount++;
        this.frame = frame;
    }
```

Notice how the field `hasNewFrame` is now set to `true` before the frame field is assigned to reference the new Frame object. That means, that if the drawing thread is waiting in the while-loop in the `takeFrame()` method, the drawing thread could exit that while-loop, and take the old Frame object. That would result in a redrawing of an old Frame, leading to a waste of resources.

Obviously, in this particular case redrawing an old frame won't make the application crash or malfunction. It just wastes CPU and GPU resources. However, in other cases such instruction reordering could make the application malfunction.

## The Java volatile Visibility Guarantee

The Java `volatile` keyword provides some visibility guarantees for when writes to, and reads of, volatile variables result in synchronization of the variable's value to and from main memory. This synchronization to and from main memory is what makes the value visible to other threads. Hence the term visibility guarantee.

In this section I will briefly cover the Java volatile visibility guarantee, and explain how instruction reordering may break the volatile visibility guarantee. That is why we also have the Java volatile happens before guarantee, to place some restrictions on instruction reordering so the volatile visibility guarantee is not broken by instruction reordering.

## The Java volatile Write Visibility Guarantee

When you write to a Java volatile variable the value is guaranteed to be written directly to main memory. Additionally, all variables visible to the thread writing to the volatile variable will also get synchronized to main memory.

To illustrate the Java volatile write visibility guarantee, look at this example:

```java
this.nonVolatileVarA = 34;
this.nonVolatileVarB = new String("Text");
this.volatileVarC    = 300;
```

This example contains two writes to non-volatile variables, and one write to a volatile variable. The example does not explicitly show which variable is declared volatile, so to be clear, imagine the variable (field, really) named `volatileVarC` is declared `volatile`.

When the third instruction in the example above writes to the volatile variable `volatileVarC`, the values of the two non-volatile variables will also be synchronized to main memory - because these variables are visible to the thread when writing to the volatile variable.

### The Java volatile Read Visibility Guarantee

When you read the value of a Java `volatile` the value is guaranteed to be read directly from memory. Furthermore, all the variables visible to the thread reading the volatile variable will also have their values refreshed from main memory.

To illustrate the Java volatile read visibility guarantee look at this example:

```java
c = other.volatileVarC;
b = other.nonVolatileB;
a = other.nonVolatileA;
```

Notice that the first instruction is a read of a `volatile` variable (`other.volatileVarC`). When `other.volatileVarC` is read in from main memory, the `other.nonVolatileB` and `other.nonVolatileA` are also read in from main memory.

## The Java Volatile Happens Before Guarantee

The Java volatile happens before guarantee sets some restrictions on instruction reordering around volatile variables. To illustrate why this guarantee is necessary, let us modify the FrameExchanger class from earlier in this tutorial to have the `hasNewFrame` variable be declared `volatile`:

```java
public class FrameExchanger  {

    private long framesStoredCount = 0:
    private long framesTakenCount  = 0;

    private volatile boolean hasNewFrame = false;

    private Frame frame = null;

        // called by Frame producing thread
    public void storeFrame(Frame frame) {
        this.frame = frame;
        this.framesStoredCount++;
        this.hasNewFrame = true;
    }

        // called by Frame drawing thread
    public Frame takeFrame() {
        while( !hasNewFrame) {
            //busy wait until new frame arrives
        }

        Frame newFrame = this.frame;
        this.framesTakenCount++;
        this.hasNewFrame = false;
        return newFrame;
    }

}
```

Now, when the `hasNewFrame` variable is set to `true`, the `frame` and `frameStoredCount` will also be synchronized to main memory. Additionally, every time the drawing thread reads the `hasNewFrame` variable in the while-loop inside the `takeFrame()` method, the `frame` and `framesStoredCount` will also be refreshed from main memory. Even `framesTakenCount` will get updated from main memory at this point.

Imagine if the Java VM reordered the instructions inside the `storeFrame()` method, like this:

```java
    // called by Frame producing thread
    public void storeFrame(Frame frame) {
        this.hasNewFrame = true;
        this.framesStoredCount++;
        this.frame = frame;
    }
```   

Now the framesStoredCount and frame fields will get synchronized to main memory when the first instruction is executed (because hasNewFrame is volatile) - which is before they have their new values assigned to them!

This means, that the drawing thread executing the `takeFrame()` method may exit the while-loop before the new value is assigned to the frame variable. Even if a new value had been assigned to the frame variable by the producing thread, there would not be any guarantee that this value would have been synchronized to main memory so it is visible for the drawing thread!

### Happens Before Guarantee for Writes to volatile Variables

As you can see, the reordering of the instructions inside `storeFrame()` method may make the application malfunction. This is where the volatile write happens before guarantee comes in - to put restrictions on what kind of instruction reordering is allowed around writes to volatile variables:

A write to a non-volatile or volatile variable that happens before a write to a volatile variable is guaranteed to happen before the write to that volatile variable.

In the case of the `storeFrame()` method that means that the two first write instructions cannot be reordered to happen after the last write to hasNewFrame, since hasNewFrame is a volatile variable.

```java
    // called by Frame producing thread
    public void storeFrame(Frame frame) {
        this.frame = frame;
        this.framesStoredCount++;
        this.hasNewFrame = true;  // hasNewFrame is volatile
    }
```

The two first instructions are not writing to volatile variables, so they can be reordered by the Java VM freely. Thus, this reordering is allowed:

```java
    // called by Frame producing thread
    public void storeFrame(Frame frame) {
        this.framesStoredCount++;
        this.frame = frame;
        this.hasNewFrame = true;  // hasNewFrame is volatile
    }
```    

This reordering does not break the code in the takeFrame() method, as the frame variable is still written to before the hasNewFrame variable is written to. The total program still works as intended.

### Happens Before Guarantee for Reads of volatile Variables

Volatile variables in Java has a similar happens before guarantee for reads of volatile variables. Only, the direction is opposite:

A read of a volatile variable will happen before any subsequent reads of volatile and non-volatile variables.

When I say the direction is different than for writes, I mean that for volatile writes all instructions before the write will remain before the volatile write. For volatile reads, all reads after the volatile read will remain after the volatile read.

Look at the following example:

```java
int a = this.volatileVarA;
int b = this.nonVolatileVarB;
int c = this.nonVolatileVarC;
```

Both of instructions 2 and 3 has to remain after the first instruction, because the first instructions reads a volatile variable. In other words, the read of the volatile variable is guaranteed to happen before the two subsequent reads of the non-volatile variables.

The last two instructions could be freely reordered among themselves, without violating the happens before guarantee of the volatile read in the first instruction. Thus, this reordering is allowed:

```java
int a = this.volatileVarA;
int c = this.nonVolatileVarC;
int b = this.nonVolatileVarB;
```

Because of the volatile read visibility guarantee, when this.volatileVarA is read from main memory, so are all other variables visible to the thread at that time. Thus, this.nonVolatileVarB and this.nonVolatileVarC are also read in from main memory at the same time. This means, that the thread that reads volatileVarA can rely on nonVolatileVarB and nonVolatileVarC to be up-to-date with main memory too.

If any of the two last instructions were to be reordered above the first volatile read instruction, the guarantee for that instruction at the time it was executed would not hold up. That's why later reads cannot be reordered to appear above a read of a volatile variable.

With regards to the takeFrame() method, the first read of a volatile variable is the read of the hasNewFrame field inside the while-loop. That means, that no read-instructions can be reordered to be located above that. In this particular case, moving any of the other read-operations above the while-loop would also break the semantics of the code, so those reorderings would not be allowed anyways.

```java
    // called by Frame drawing thread
    public Frame takeFrame() {
        while( !hasNewFrame) {
            //busy wait until new frame arrives
        }

        Frame newFrame = this.frame;
        this.framesTakenCount++;
        this.hasNewFrame = false;
        return newFrame;
    }
```   

## The Java Synchronized Visibility Guarantee

Java synchronized blocks provide visibility guarantees that are similar to those of Java volatile variables. I will explain the Java synchronized visibility guarantee briefly.

### Java Synchronized Entry Visibility Guarantee

When a thread enters a synchronized block, all variables visible to the thread are refreshed from main memory.

### Java Synchronized Exit Visibility Guarantee

When a thread exits a synchronized block, all variables visible to the thread are written back to main memory.

### Java Synchronized Visibility Example

Look at this ValueExchanger class:

```java
public class ValueExchanger {
    private int valA;
    private int valB;
    private int valC;

    public void set(Values v) {
        this.valA = v.valA;
        this.valB = v.valB;

        synchronized(this) {
            this.valC = v.valC;
        }
    }

    public void get(Values v) {
        synchronized(this) {
            v.valC = this.valC;
        }
        v.valB = this.valB;
        v.valA = this.valA;
    }
}
```

Notice the two synchronized blocks inside the set() and get() method. Notice how the blocks are placed last and first in the two methods.

In the set() method the synchronized block at the end of the method will force all the variables to be synchronized to main memory after being updated. This flushing of the variable values to main memory happens when the thread exits the synchronized block. That is the reason it has been placed last in the method - to guarantee that all updated variable values are flushed to main memory.

In the get() method the synchronized block is placed at the beginning of the method. When the thread calling get() enters the synchronized block, all variables are re-read in from main memory. That is why this synchronized block is placed at the beginning of the method - to guarantee that all variables are refreshed from main memory before they are read.

## Java Synchronized Happens Before Guarantee

Java synchronized blocks provide two happens before guarantees: One guarantee related to the beginning of a synchronized block, and another guarantee related to the end of a synchronized block. I will cover both in the following sections.

### Java Synchronized Block Beginning Happens Before Guarantee

The beginning of a Java synchronized block provides the visibility guarantee (mentioned earlier in this tutorial), that when a thread enters a synchronized block all variables visible to the thread will be read in (refreshed from) main memory.

To be able to uphold that guarantee, a set of restrictions on instruction reordering are necessary. To illustrate why, I will use the get() method of the ValueExchanger shown earlier:

```java
    public void get(Values v) {
        synchronized(this) {
            v.valC = this.valC;
        }
        v.valB = this.valB;
        v.valA = this.valA;
    }
```

As you can see, the synchronized block at the beginning of the method will guarantee that all of the variables this.valC, this.valB and this.valA are refreshed (read in) from main memory. The following reads of these variables will then use the latest value.

For this to work, none of the reads of the variables can be reordered to appear before the beginning of the synchronized block. If a read of a variable was reordered to appear before the beginning of the synchronized block, you would lose the guarantee of the variable values being refreshed from main memory. That would be the case with the following, unpermitted reordering of the instructions:

```java
    public void get(Values v) {
        v.valB = this.valB;
        v.valA = this.valA;
        synchronized(this) {
            v.valC = this.valC;
        }
    }
```

### Java Synchronized Block End Happens Before Guarantee

The end of a synchronized block provides the visibility guarantee that all changed variables will be written back to main memory when the thread exits the synchronized block.

To be able to uphold that guarantee, a set of restrictions on instruction reordering are necessary. To illustrate why, I will use the set() method of the ValueExchanger shown earlier:

```java
    public void set(Values v) {
        this.valA = v.valA;
        this.valB = v.valB;

        synchronized(this) {
            this.valC = v.valC;
        }
    }
```

As you can see, the synchronized block at the end of the method will guarantee that all of the changed variables this.valA, this.valB and this.valC will be written back to (flushed) to main memory when the thread calling set() exits the synchronized blocks.

For this to work, none of the writes to the variables can be reordered to appear after the end of the synchronized block. If the writes to the variables were reordered to to appear after the end of the synchronized block, you would lose the guarantee of the variable values being written back to main memory. That would be the case in the following, unpermitted reordering of the instructions:

```java
    public void set(Values v) {
        synchronized(this) {
            this.valC = v.valC;
        }
        this.valA = v.valA;
        this.valB = v.valB;
    }
```

The End.

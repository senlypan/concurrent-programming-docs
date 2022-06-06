# Multithreading Benefits

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.concurrent.01-multithreading-benefits-en&left_color=blue&right_color=red)

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/benefits.html  Update: 2022-02-23

The most significant benefits of multithreading are:

- Better CPU utilization.
- Simpler program design in some situations.
- More responsive programs.
- More fair division of CPU resources between different tasks.

## Better CPU Utilization

Imagine an application that reads and processes files from the local file system. Lets say that reading a file from disk takes 5 seconds and processing it takes 2 seconds. Processing two files then takes

```text
  5 seconds reading file A
  2 seconds processing file A
  5 seconds reading file B
  2 seconds processing file B
-----------------------
 14 seconds total
```

When reading the file from disk most of the CPU time is spent waiting for the disk to read the data. The CPU is pretty much idle during that time. It could be doing something else. By changing the order of the operations, the CPU could be better utilized. Look at this ordering:

```text 
  5 seconds reading file A
  5 seconds reading file B + 2 seconds processing file A
  2 seconds processing file B
-----------------------
 12 seconds total
```

The CPU waits for the first file to be read. Then it starts the read of the second file. While the second file is being read in by the IO components of the computer, the CPU processes the first file. Remember, while waiting for the file to be read from disk, the CPU is mostly idle.

In general, the CPU can be doing other things while waiting for IO. It doesn't have to be disk IO. It can be network IO as well, or input from a user at the machine. Network and disk IO is often a lot slower than CPU's and memory IO.

## Simpler Program Design

If you were to program the above ordering of reading and processing by hand in a singlethreaded application, you would have to keep track of both the read and processing state of each file. Instead you can start two threads that each just reads and processes a single file. Each of these threads will be blocked while waiting for the disk to read its file. While waiting, other threads can use the CPU to process the parts of the file they have already read. The result is, that the disk is kept busy at all times, reading from various files into memory. This results in a better utilization of both the disk and the CPU. It is also easier to program, since each thread only has to keep track of a single file.

## More Responsive Programs

Another common goal for turning a singlethreaded application into a multithreaded application is to achieve a more responsive application. Imagine a server application that listens on some port for incoming requests. when a request is received, it handles the request and then goes back to listening. The server loop is sketched below:

```java

 while(server is active){
    listen for request
    process request
  }

```

If the request takes a long time to process, no new clients can send requests to the server for that duration. Only while the server is listening can requests be received.

An alternate design would be for the listening thread to pass the request to a worker thread, and return to listening immediatedly. The worker thread will process the request and send a reply to the client. This design is sketched below:

```java

 while(server is active){
    listen for request
    hand request to worker thread
  }

```

This way the server thread will be back at listening sooner. Thus more clients can send requests to the server. The server has become more responsive.

The same is true for desktop applications. If you click a button that starts a long task, and the thread executing the task is the thread updating the windows, buttons etc., then the application will appear unresponsive while the task executes. Instead the task can be handed off to a worker thread. While the worker thread is busy with the task, the window thread is free to respond to other user requests. When the worker thread is done it signals the window thread. The window thread can then update the application windows with the result of the task. The program with the worker thread design will appear more responsive to the user.

## More Fair Distribution of CPU Resources

Imagine a server that is receiving requests from clients. Imagine then, that one of the clients sends a request that takes a long time to process - e.g. 10 seconds. If the server processed all tasks using a single thread, then all requests following the request that was slow to process would be forced to wait until the full request has been processed.

By dividing the CPU time between multiple threads and switching between the threads, then the CPU can share its execution time more fairly between several requests. Then even if one of the requests is slow, other requests that are faster to process can be executed concurrently with the slower request. Of course this means that executing the slow request will be even slower, since it will not have the CPU solely allocated to processing just it. However, the other requests will have to wait shorter time to be processed, because they do not have to wait for the slow tasks to finish before they can be processed. In the event that there is only the slow request to process, then the CPU can still be solely allocated to the slow task.

The End.
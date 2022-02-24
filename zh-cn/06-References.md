# Java Concurrency References

> Author: Jakob Jenkov
>
> Link: http://tutorials.jenkov.com/java-concurrency/references.html  Update: 2022-02-24

## ⛔抱歉，本文暂无中文翻译，持续更新中
?> ❤️ 您也可以参与翻译，快来提交 [issue](https://github.com/senlypan/concurrent-programming-docs/issues) 或投稿参与吧~

From time to time I get asked about what books and articles I have read before writing this Java concurrency tutorial. Concurrency is tricky so people often like to be able to double check my writing against other sources. Therefore I have collected this list of references of concurrency related books and articles that I have used in the writing of this Java concurrency tutorial.

## Books

[Java Concurrency in Practice]()

This is the newest book on Java concurrency. It is from 2006 so it is a bit dated in some ways. For instance, it does not cover asynchronous architectures much (which are getting popular now in 2015). It is a decent book on Java concurrency. By far the best book on the java.util.concurrent package in Java 5 and forward.

[Seven Concurrency Models in Seven Weeks](https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601/)

This book is newer (from 2014) and covers different concurrency models - not just the traditional Threads, Shared Memory and Locks model.

[Concurrent Programming in Java - 1st + 2nd Ed](https://www.amazon.com/Seven-Concurrency-Models-Weeks-Programmers/dp/1937785653/)

This book (in two editions) was a good primer on Java concurrency back in the day. The book is from 1999, so a lot has happened in Java and with concurrency models since then.

[Taming Java Threads](https://www.amazon.com/Concurrent-Programming-Java%C2%99-Principles-Pattern/dp/0201310090/)

This was also a good primer on Java threading back in the day. The book is from 2000, so like with Doug Lea's book, a lot has changed since.

[Java Threads](https://www.amazon.com/Java-Threads-Scott-Oaks/dp/0596007825/)

A decent book on Java threading from 2004. Most of the material in this book is covered in other books though, so I didn't read it too closely.

## Articles

https://www.cise.ufl.edu/tr/DOC/REP-1991-12.pdf

Good article on non-blocking concurrency algorithms (from 1991).

## Other Resources

https://lmax-exchange.github.io/disruptor/

The LMAX Disrupter concurrent data structure (a single reader, single writer queue-like structure with high concurrency).

The End.
# Java基础面试专题-线程和IO

## 1. 谈谈你对线程的理解

&emsp;&emsp;Java线程是Java程序中的执行单元。一个Java程序可以同时运行多个线程，每个线程可以独立执行不同的任务。线程的执行是并发的，即多个线程可以同时执行。

### 1.1 线程的特点

&emsp;&emsp;Java中的线程有如下的特点

1. 轻量级：线程的创建和销毁的开销相对较小，可以创建大量的线程。
2. 共享内存：多个线程可以共享同一块内存区域，这使得线程之间可以方便地进行数据通信。
3. 独立调度：每个线程的执行是由操作系统进行调度的，线程的调度是非确定性的，也就是说无法预测线程的执行顺序。

### 1.2 线程的创建方式

1. 继承Thread类：创建一个继承自Thread类的子类，并重写run()方法，在run()方法中定义线程的任务。然后通过调用子类的start()方法来启动线程。
2. 实现Runnable接口：创建一个实现了Runnable接口的类，并实现其run()方法，在run()方法中定义线程的任务。然后通过创建Thread对象，将实现了Runnable接口的对象作为参数传入，并调用Thread对象的start()方法来启动线程。

### 1.3 线程的状态

&emsp;&emsp;线程的状态也是面试中会问的比较多的。

1. 新建状态（New）：线程对象被创建后，但还没有调用start()方法时的状态。
2. 就绪状态（Runnable）：线程对象调用start()方法后进入就绪状态，表示线程可以被调度执行。
3. 运行状态（Running）：线程被调度执行后进入运行状态。
4. 阻塞状态（Blocked）：线程在执行过程中可能因为某些原因被阻塞，例如等待输入输出、线程休眠等。
5. 结束状态（Terminated）：线程执行完任务后进入结束状态。

图例如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1691558059086/53a97a029fe64bc38e1fd3e47c4ab736.png)

### 1.4 线程间的通信

Java中线程间通信的方式有以下几种：

1. wait()和notify()方法：wait()方法使线程进入等待状态，直到其他线程调用notify()或notifyAll()方法将其唤醒。notify()方法唤醒一个等待中的线程，notifyAll()方法唤醒所有等待中的线程。
2. wait(long timeout)和notify()方法：wait(long timeout)方法使线程进入等待状态，直到其他线程调用notify()方法将其唤醒，或者等待时间超过指定的timeout时间。notify()方法唤醒一个等待中的线程。
3. join()方法：join()方法使一个线程等待另一个线程执行完毕。当一个线程调用另一个线程的join()方法时，当前线程将被阻塞，直到另一个线程执行完毕。
4. Lock和Condition接口：Lock接口提供了比synchronized关键字更灵活的锁机制，Condition接口提供了更灵活的等待/通知机制。通过Lock接口的lock()方法获取锁，unlock()方法释放锁；通过Condition接口的await()方法使线程等待，signal()方法唤醒一个等待中的线程，signalAll()方法唤醒所有等待中的线程。
5. BlockingQueue阻塞队列：BlockingQueue是一个支持阻塞操作的队列，当队列为空时，获取元素的线程将被阻塞，直到队列中有可用元素；当队列满时，插入元素的线程将被阻塞，直到队列有空闲位置。

## 2.谈谈对TheadLocal的理解

&emsp;&emsp;ThreadLocal可以理解为线程本地变量，他会在每个线程都创建一个副本，那么在线程之间访问内部副本变量就行了，做到了线程之间互相隔离，相比于synchronized的做法是用空间来换时间。

&emsp;&emsp;ThreadLocal有一个静态内部类ThreadLocalMap，ThreadLocalMap又包含了一个Entry数组，Entry本身是一个弱引用，他的key是指向ThreadLocal的弱引用，Entry具备了保存key value键值对的能力。

&emsp;&emsp;弱引用的目的是为了防止内存泄露，如果是强引用那么ThreadLocal对象除非线程结束否则始终无法被回收，弱引用则会在下一次GC的时候被回收。

&emsp;&emsp;但是这样还是会存在内存泄露的问题，假如key和ThreadLocal对象被回收之后，entry中就存在key为null，但是value有值的entry对象，但是永远没办法被访问到，同样除非线程结束运行。

&emsp;&emsp;但是只要ThreadLocal使用恰当，在使用完之后调用remove方法删除Entry对象，实际上是不会出现这个问题的。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1691558059086/39212b6f92584828986ffba5955a4db2.png)

## 3. Synchronized和lock的区别

&emsp;&emsp;Synchronized和Lock都是用于实现线程之间的同步的机制，但它们之间有一些区别。

1. 锁的类型：Synchronized是Java中的关键字，它只能用于同步代码块或方法。而Lock是一个接口，Java提供了多种实现该接口的锁，如ReentrantLock、ReadWriteLock等。
2. 使用方式：Synchronized是隐式锁，它的获取和释放由JVM自动管理，无需手动控制。而Lock是显式锁，需要手动调用lock()方法获取锁，并在合适的地方调用unlock()方法释放锁。
3. 可中断性：在获取锁时，如果线程无法获取到锁，Synchronized会一直等待，直到获取到锁。而Lock提供了可中断性，即可以在等待获取锁的过程中，中断线程的等待。
4. 公平性：Synchronized不保证线程获取锁的公平性，即无法保证等待时间最长的线程优先获取锁。而Lock可以通过构造函数指定锁的公平性，即等待时间最长的线程会优先获取锁。
5. 条件变量：Lock提供了Condition接口，可以通过该接口实现线程之间的等待/通知机制。而Synchronized没有直接提供类似于Condition的功能，需要借助于Object的wait()、notify()和notifyAll()方法来实现。

&emsp;&emsp;总的来说，Synchronized是一种简单、易用的同步机制，适用于大多数情况下。而Lock提供了更多的灵活性和功能，适用于一些复杂的同步场景。在性能方面，Lock通常比Synchronized更加高效，但使用Lock需要手动管理锁的获取和释放，容易出错。

https://www.processon.com/view/link/64d350632a06f6607227938c

## 4.谈谈你对IO的理解

### 4.1 Java基础知识

&emsp;&emsp;Java IO（Input/Output）是Java编程语言中用于处理输入和输出的一组类和接口。它提供了一种在Java程序中读取和写入数据的方法。

Java IO包括两个主要的部分：

* 字节流:以字节为单位进行操作,字节流适用于处理二进制数据.
* 字符流。以字符为单位进行操作,而字符流适用于处理文本数据。

&emsp;&emsp;Java IO的核心类是InputStream和OutputStream，它们分别用于从输入源读取数据和向输出目标写入数据。另外，Reader和Writer类是用于读取和写入文本数据的字符流的基类。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1691558059086/a51a6389217441b3bacd86d1fa68451c.png)

### 4.2 设计模式

&emsp;&emsp;在IO的设计中其实也穿插了很多设计模式的应用。这块也是可以在面试的时候很好的和面试官畅聊的

* 装饰器模式
* 观察者模式
* 适配器模式
* 工厂模式

### 4.3 Java IO模型

&emsp;&emsp; IO模型这块是相对比较有难度的内容。我们可以从其中的一个点作为突破口来和面试官沟通。比如

* BIO（Blocking I/O）**BIO 属于同步阻塞 IO 模型 。**
* NIO（Non-blocking/New I/O）Java 中的 NIO 可以看作是 **I/O 多路复用模型**。也有很多人认为，Java 中的 NIO 属于同步非阻塞 IO 模型。
* AIO（Asynchronous I/O）**异步 IO 模型**

这三者的介绍，资料分享链接：

[IO详细完整笔记](https://www.processon.com/view/link/64a8d8d01906b3205606463f)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1691558059086/ec55e6ae6e8442a29899346f961b8ac4.png)

https://www.processon.com/view/link/64d352e1f136581c9f1a72a6

# 1.Lock源码分析

在上一篇文章中我们讲到了如何使用关键字synchronized来实现同步访问。本文我们继续来探讨这个问题，从Java 5之后，在java.util.concurrent.locks包下提供了另外一种方式来实现同步访问，那就是Lock。

也许有朋友会问，既然都可以通过synchronized来实现同步访问了，那么为什么还需要提供Lock？这个问题将在下面进行阐述。本文先从synchronized的缺陷讲起，然后再讲述java.util.concurrent.locks包下常用的有哪些类和接口，最后讨论以下一些关于锁的概念方面的东西

## synchronized缺陷 {#synchronized%E7%BC%BA%E9%99%B7}

前面我们说过synchronized的线程释放锁的情况有两种：

1.代码块或者同步方法执行完毕

2.代码块或者同步方法出现异常有jvm自动释放锁

从上面的synchronized释放锁可以看出，**只有synchronized代码块执行完毕或者异常才会释放，如果代码块中的程序因为IO原因阻塞了，那么线程将永远不会释放锁，但是此时另外的线程还要执行其他的程序，极大的影响了程序的执行效率。**

现在我们需要一种机制能够让线程不会一直无限的等待下去，能够响应中断，这个通过lock就可以办到 另外**如果有一个程序，包含多个读线程和一个写线程，我们可以知道synchronized只能一个一个线程的执行，但是我们需要多个读线程同时进行读，那么使用synchronized肯定是不行的，但是我们使用lock同样可以办到。**

## Lock

锁，用于并发编程，该接口声明了申请锁和释放锁的方法

查看API可知，Lock是一个接口，因此是不可以直接创建对象的，但是我们可以利用其实现的类来创建对象，这个先不着急，我们先看看Lock类到底实现了什么方法,具体的实现我们将会在介绍其实现的类的时候再详细的讲解



Lock接口中定义了对锁的各种操作

```
public interface Lock {

    // 不响应中断的获取锁
    void lock();

    // 响应中断的获取锁
    void lockInterruptibly() throws InterruptedException;

    // 尝试非阻塞的获取锁，true为获取到锁，false为没有获取到锁
    boolean tryLock();

    // 超时获取锁，以下情况会返回：时间内获取到了锁，时间内被中断，时间到了没有获取到锁
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    // 释放锁
    void unlock();

    // 创建一个condition
    Condition newCondition();
}
```




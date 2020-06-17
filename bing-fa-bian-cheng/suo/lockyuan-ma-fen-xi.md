# 1.Lock源码分析

在上一篇文章中我们讲到了如何使用关键字synchronized来实现同步访问。本文我们继续来探讨这个问题，从Java 5之后，在java.util.concurrent.locks包下提供了另外一种方式来实现同步访问，那就是Lock。 也许有朋友会问，既然都可以通过synchronized来实现同步访问了，那么为什么还需要提供Lock？这个问题将在下面进行阐述。本文先从synchronized的缺陷讲起，然后再讲述java.util.concurrent.locks包下常用的有哪些类和接口，最后讨论以下一些关于锁的概念方面的东西

## synchronized缺陷 {#synchronized%E7%BC%BA%E9%99%B7}

## Lock

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




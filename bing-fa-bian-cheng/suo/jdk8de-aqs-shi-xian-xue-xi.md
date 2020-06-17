# 1.JDK8的AQS实现学习

## 1.1.带着问题学习

1.AQS是什么鬼东西  
2.AQS是怎么实现的

## 1.2.基本介绍

```
// 同步队列，是一个带头结点的双向链表，用于实现锁的语义
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements Serializable {
......
}
```

所谓AQS，指的是AbstractQueuedSynchronizer，它提供了一种实现阻塞锁和一系列依赖FIFO等待队列的同步器的框架，ReentrantLock、Semaphore、CountDownLatch、CyclicBarrier等并发类均是基于AQS来实现的，具体用法是通过继承AQS实现其模板方法，然后将子类作为同步组件的内部类。

10431632-7d2aa48b9b217bbe.webp

AQS维护了一个volatile语义\(支持多线程下的可见性\)的共享资源变量state和一个FIFO线程等待队列\(多线程竞争state被阻塞时会进入此队列\)。

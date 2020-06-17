# 1.JDK8的AQS实现源码学习

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

所谓AQS，指的是**AbstractQueuedSynchronizer**，它提供了一种实现**阻塞锁和一系列依赖FIFO等待队列的同步器的框架**，ReentrantLock、Semaphore、CountDownLatch、CyclicBarrier等并发类均是基于AQS来实现的，具体用法是通过继承AQS实现其模板方法，然后将子类作为同步组件的内部类。

## 1.3.基本框架

AQS是一个同步器，设计模式是模板模式。

  


核心数据结构：双向链表 + state\(锁状态\)

  


底层操作：CAS

![](/static/image/10431632-7d2aa48b9b217bbe.webp)

**AQS维护了一个volatile语义\(支持多线程下的可见性\)的共享资源变量state和一个FIFO线程等待队列\(多线程竞争state被阻塞时会进入此队列\)。**

```
注释：最重要的就是搞清楚state和FIFO线程等待队列是怎么来实现这个同步器的框架
```

![](/static/image/微信截图_20200617174450.png)

### CLH队列\(FIFO\)

### 资源的共享方式分为2种

# 2.总结

J.U.C是基于AQS实现的，AQS是一个同步器，设计模式是模板模式。

核心数据结构：双向链表 + state\(锁状态\)

底层操作：CAS


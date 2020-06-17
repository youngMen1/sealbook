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

### AbstractQueuedSynchronizer类

![](/static/image/微信截图_20200617174450.png)

### Node类

![](/static/image/441222012158451512152485.webp)

### state

首先说一下共享资源变量state，它是int数据类型的，其访问方式有3种：

* getState\(\)
* setState\(int newState\)
* compareAndSetState\(int expect, int update\)

上述3种方式均是原子操作，其中compareAndSetState\(\)的实现依赖于Unsafe的compareAndSwapInt\(\)方法。

![](/static/image/10431632-7d2aa48b9b217bbe.webp)

**AQS维护了一个volatile语义\(支持多线程下的可见性\)的共享资源变量state和一个FIFO线程等待队列\(多线程竞争state被阻塞时会进入此队列\)。**

```
注释：最重要的就是搞清楚state和FIFO线程等待队列是怎么来实现这个同步器的框架
```

### CLH队列\(FIFO\)

AQS是通过内部类Node来实现FIFO队列的，源代码解析如下：

```
static final class Node {

    // 表明节点在共享模式下等待的标记
    static final Node SHARED = new Node();
    // 表明节点在独占模式下等待的标记
    static final Node EXCLUSIVE = null;

    // 表征等待线程已取消的
    static final int CANCELLED =  1;
    // 表征需要唤醒后续线程
    static final int SIGNAL    = -1;
    // 表征线程正在等待触发条件(condition)
    static final int CONDITION = -2;
    // 表征下一个acquireShared应无条件传播
    static final int PROPAGATE = -3;

    /**
     *   SIGNAL: 当前节点释放state或者取消后，将通知后续节点竞争state。
     *   CANCELLED: 线程因timeout和interrupt而放弃竞争state，当前节点将与state彻底拜拜
     *   CONDITION: 表征当前节点处于条件队列中，它将不能用作同步队列节点，直到其waitStatus被重置为0
     *   PROPAGATE: 表征下一个acquireShared应无条件传播
     *   0: None of the above
     */
    volatile int waitStatus;

    // 前继节点
    volatile Node prev;
    // 后继节点
    volatile Node next;
    // 持有的线程
    volatile Thread thread;
    // 链接下一个等待条件触发的节点
    Node nextWaiter;

    // 返回节点是否处于Shared状态下
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    // 返回前继节点
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    // Shared模式下的Node构造函数
    Node() {  
    }

    // 用于addWaiter
    Node(Thread thread, Node mode) {  
        this.nextWaiter = mode;
        this.thread = thread;
    }

    // 用于Condition
    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

### 资源的共享方式分为2种

# 2.总结

J.U.C是基于AQS实现的，AQS是一个同步器，设计模式是模板模式。

核心数据结构：双向链表 + state\(锁状态\)

底层操作：CAS


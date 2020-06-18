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

所谓AQS，指的是**AbstractQueuedSynchronizer**，它提供了一种实现**阻塞锁和一系列依赖FIFO等待队列的同步器的框架**，**ReentrantLock、Semaphore、CountDownLatch、CyclicBarrier等并发类均是基于AQS来实现的**，具体用法是通过继承AQS实现其模板方法，然后将子类作为同步组件的内部类。

注释：后面学习**ReentrantLock、Semaphore、CountDownLatch、CyclicBarrier的时候会看到均是基于AQS来实现的这里不做拓展。**

## 1.3.基本框架

AQS是一个同步器，设计模式是模板模式。

核心数据结构：双向链表 + state\(锁状态\)

底层操作：CAS

AQS基本框架如下图所示：  
![](/static/image/10431632-7d2aa48b9b217bbe.webp)

### 1.3.1AbstractQueuedSynchronizer类

![](/static/image/微信截图_20200617174450.png)

### 1.3.2.Node内部类

![](/static/image/441222012158451512152485.webp)

### 1.3.4.state

首先说一下共享资源变量state，它是int数据类型的，其访问方式有3种：

* getState\(\)
* setState\(int newState\)
* compareAndSetState\(int expect, int update\)

重入锁计数/许可证数量，在不同的锁中，使用方式有所不同

上述3种方式均是原子操作，其中compareAndSetState\(\)的实现依赖于Unsafe的compareAndSwapInt\(\)方法。

**AQS维护了一个volatile语义\(支持多线程下的可见性\)的共享资源变量state和一个FIFO线程等待队列\(多线程竞争state被阻塞时会进入此队列\)。**

```
// 重入锁计数/许可证数量，在不同的锁中，使用方式有所不同
private volatile int state;

// 具有内存读可见性语义
protected final int getState() {
    return state;
}

// 具有内存写可见性语义
protected final void setState(int newState) {
    state = newState;
}

// 具有内存读/写可见性语义
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

AQS中的int类型的state值，各种锁就是通过CAS（乐观锁）去修改state的值。lock的基本操作还是通过乐观锁来实现的。

### 1.3.5.CLH队列\(FIFO\)

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

```
    private transient volatile Node head;   // 【|同步队列|】的头结点
    private transient volatile Node tail;   // 【|同步队列|】的尾结点



    // 前继节点
    volatile Node prev;
    // 后继节点
    volatile Node next;
```

最后我们可以发现锁的存储结构就两个东西:"**双向链表**" + "**waitStatus的int类型状态**"。

需要注意的是，他们的变量都被"`transient`和`volatile`修饰。

还可以看到，waitStatus非负的时候，表征不可用，正数代表处于等待状态，所以waitStatus只需要检查其正负符号即可，不用太多关注特定值。

### 1.3.6.资源的共享方式分为2种（后面细讲）

#### 独占式\(Exclusive\)

只有单个线程能够成功获取资源并执行，如ReentrantLock。

获取资源

* public final void acquire\(int arg\)  申请独占锁，允许阻塞带有中断标记的线程（会先将其标记清除）

释放资源

* public final boolean release\(int arg\)  释放锁，如果锁已被完全释放，则唤醒后续的阻塞线程。返回值表示本次操作后锁是否自由

#### 共享式\(Shared\)

多个线程可成功获取资源并执行，如Semaphore/CountDownLatch等。

获取资源

* public final void acquireShared\(int arg\) 申请共享锁，若获取成功则直接返回，若失败，则进入等待队列，执行自旋获取资源。

释放资源

* public final boolean releaseShared\(int arg\) 释放锁，并唤醒排队的结点。

AQS需要子类复写的方法均没有声明为abstract，目的是避免子类需要强制性覆写多个方法，因为一般自定义同步器要么是独占方法，要么是共享方法，只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。

当然，AQS也支持子类同时实现独占和共享两种模式，如ReentrantReadWriteLock。

AQS在独占和共享两种模式下，在acquire\(\)和acquireShared\(\)方法中，线程在阻塞过程中均是忽略中断的。

# 2.总结

AQS指的是AbstractQueuedSynchronizer

**J.U.C是基于AQS实现的**，AQS是一个同步器，设计模式是模板模式。

核心数据结构：双向链表 + state\(锁状态\)

底层操作：CAS

# 3.参考

非常感谢康建伟分享的jdk源码笔记

```
https://github.com/kangjianwei/LearningJDK
```

浅谈Java的AQS：

```
https://www.jianshu.com/p/0f876ead2846
```

[一文带你理解Java中Lock的实现原理](https://links.jianshu.com/go?to=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI3NzE0NjcwMg%3D%3D%26mid%3D2650122072%26idx%3D1%26sn%3D63690ad2cbf2b5390c3d8e1953ffbacf%26chksm%3Df36bba79c41c336fbea8b56289fc2a71e829042f6c3616e3ba051c2542b48f0a3936e3d852f6%26mpshare%3D1%26scene%3D1%26srcid%3D0225xcUOCP6bBS8aCrcd1jBd%23rd)


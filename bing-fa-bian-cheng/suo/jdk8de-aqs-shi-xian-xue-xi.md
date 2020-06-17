# 1.JDK8的AQS实现学习

```
// 同步队列，是一个带头结点的双向链表，用于实现锁的语义
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements Serializable {
......
}
```

所谓AQS，指的是AbstractQueuedSynchronizer，它提供了一种实现阻塞锁和一系列依赖FIFO等待队列的同步器的框架，ReentrantLock、Semaphore、CountDownLatch、CyclicBarrier等并发类均是基于AQS来实现的，具体用法是通过继承AQS实现其模板方法，然后将子类作为同步组件的内部类。

  


  


作者：安中古天乐

  


链接：https://www.jianshu.com/p/0f876ead2846

  


来源：简书

  


著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


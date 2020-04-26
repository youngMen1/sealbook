## 基于Zookeeper实现的分布式锁\(可重入\)

# 1.基本介绍

## 1.1.**基于ZooKeeper分布式锁的流程** {#基于zookeeper分布式锁的流程}

* 在zookeeper指定节点（locks）下创建临时顺序节点node\_n
* 获取locks下所有子节点children
* 对子节点按节点自增序号从小到大排序
* 判断本节点是不是第一个子节点，若是，则获取锁；若不是，则监听比该节点小的那个节点的删除事件
* 若监听事件生效，则回到第二步重新进行判断，直到获取到锁

## **具体实现** {#具体实现}

###### **下面就具体使用java和zookeeper实现分布式锁，操作zookeeper使用的是apache提供的zookeeper的包。**

* ###### **通过实现Watch接口，实现process\(WatchedEvent event\)方法来实施监控，使CountDownLatch来完成监控，在等待锁的时候使用CountDownLatch来计数，等到后进行countDown，停止等待，继续运行。**
* ###### **以下整体流程基本与上述描述流程一致，只是在监听的时候使用的是CountDownLatch来监听前一个节点。**

# 2.怎么使用

# 3.总结

# 4.参考

分布式锁与实现（二）——基于ZooKeeper实现：

[https://www.cnblogs.com/liuyang0/p/6800538.html](https://www.cnblogs.com/liuyang0/p/6800538.html)


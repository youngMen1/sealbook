## 基于Zookeeper实现的分布式锁\(可重入\)

# 1.基本介绍

下面就具体使用java和zookeeper实现分布式锁，操作zookeeper使用的是apache提供的zookeeper的包。

* 通过实现Watch接口，实现process\(WatchedEvent event\)方法来实施监控，使CountDownLatch来完成监控，在等待锁的时候使用CountDownLatch来计数，等到后进行countDown，停止等待，继续运行。
* 以下整体流程基本与上述描述流程一致，只是在监听的时候使用的是CountDownLatch来监听前一个节点。

# 2.怎么使用

# 3.总结

# 4.参考

分布式锁与实现（二）——基于ZooKeeper实现：

[https://www.cnblogs.com/liuyang0/p/6800538.html](https://www.cnblogs.com/liuyang0/p/6800538.html)


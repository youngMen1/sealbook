## 

# 1.基本介绍

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

ZooKeeper的架构通过冗余服务实现高可用性。因此，如果第一次无应答，客户端就可以询问另一台ZooKeeper主机。ZooKeeper节点将它们的数据存储于一个分层的命名空间，非常类似于一个文件系统或一个前缀树结构。客户端可以在节点读写，从而以这种方式拥有一个共享的配置服务。更新是全序的。

## 1.1.**基于ZooKeeper分布式锁的流程** {#基于zookeeper分布式锁的流程}

* 在zookeeper指定节点（locks）下创建临时顺序节点node\_n
* 获取locks下所有子节点children
* 对子节点按节点自增序号从小到大排序
* 判断本节点是不是第一个子节点，若是，则获取锁；若不是，则监听比该节点小的那个节点的删除事件
* 若监听事件生效，则回到第二步重新进行判断，直到获取到锁

## 1.2.**具体实现** {#具体实现}

###### **下面就具体使用java和zookeeper实现分布式锁，操作zookeeper使用的是apache提供的zookeeper的包。**

* ###### **通过实现Watch接口，实现process\(WatchedEvent event\)方法来实施监控，使CountDownLatch来完成监控，在等待锁的时候使用CountDownLatch来计数，等到后进行countDown，停止等待，继续运行。**
* ###### **以下整体流程基本与上述描述流程一致，只是在监听的时候使用的是CountDownLatch来监听前一个节点。**

# 2.怎么使用

## 基于Zookeeper实现的分布式锁\(可重入\)

# 3.总结

# 4.参考

分布式锁与实现（二）——基于ZooKeeper实现：

[https://www.cnblogs.com/liuyang0/p/6800538.html](https://www.cnblogs.com/liuyang0/p/6800538.html)


# 1.基本介绍

## 1.1.[Redis的三个框架：Jedis,Redisson,Lettuce](https://www.cnblogs.com/liyan492/p/9858548.html)

Jedis api 在线网址：[http://tool.oschina.net/uploads/apidocs/redis/clients/jedis/Jedis.html](http://tool.oschina.net/uploads/apidocs/redis/clients/jedis/Jedis.html)

redisson 官网地址：[https://redisson.org/](https://redisson.org/)

redisson git项目地址：[https://github.com/redisson/redisson](https://github.com/redisson/redisson)

lettuce 官网地址：[https://lettuce.io/](https://lettuce.io/)

lettuce git项目地址：[https://github.com/lettuce-io/lettuce-core](https://github.com/lettuce-io/lettuce-core)

### 1.1.1.概念：

* Jedis：是Redis的Java实现客户端，提供了比较全面的Redis命令的支持，

* Redisson：实现了分布式和可扩展的Java数据结构。

* _Lettuce：高级Redis客户端，用于线程安全同步，异步和响应使用，支持集群，Sentinel，管道和编码器。_

_**优点：**_

_　　Jedis：比较全面的提供了Redis的操作特性_

_　　Redisson：促使使用者对Redis的关注分离，提供很多分布式相关操作服务，例如，分布式锁，分布式集合，可通过Redis支持延迟队列_

_　　Lettuce：主要在一些分布式缓存框架上使用比较多_

_**可伸缩：**_

Jedis：使用阻塞的I/O，且其方法调用都是同步的，程序流需要等到sockets处理完I/O才能执行，不支持异步。Jedis客户端实例不是线程安全的，所以需要通过连接池来使用Jedis。

Redisson：基于Netty框架的事件驱动的通信层，其方法调用是异步的。Redisson的API是线程安全的，所以可以操作单个Redisson连接来完成各种操作

_Lettuce：基于Netty框架的事件驱动的通信层，其方法调用是异步的。Lettuce的API是线程安全的，所以可以操作单个Lettuce连接来完成各种操作_

# 2.结论：

_建议使用：Jedis + Redisson_

SpringBoot项目上使用RedisTemplate会好一些，现在SpringBoot2.0默认也是使用Lettuce的


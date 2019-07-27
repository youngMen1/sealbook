## memcached和memcacheq有什么关系和联系，区别？

#### memcacheq:用于作队列

memcacheq依赖于libevent和BerkleyDB

BerkleyDB用于持久化存储队列的数据。 这样在MEMCACHEQ崩溃或者服务器挂掉的时候，

不至于造成数据的丢失

#### memcached:现在主要用于作缓存

## 什么是memcacheq?

Memcacheq是一个基于memcacheq协议、BDB持久数据存储、高性能轻量级队列服务程序。q是queue的意思，是一个列队。

## Memcacheq的特点：

1. 简单高效，基于memcache协议，这意味着只要客户端支持memcache协议即可使用。

2. 队列数据存储与BDB，持久保存。

3. 并发性能好。

4. 支持多条队列。

Memcacheq依赖于Brekeley DB和libevent。Berkeley DB 用于持久化存储队列的数据，避免在memcacheq崩溃或这服务器当掉时候，不至于数据丢失。对于并发量较高的web环境，特别是数据库写入操作过多的情景，使用队列可大大缓解因并发问题造成的数据库锁死问题。

Memcacheq的最大优势是：它是基于memcached开发的，可以通过各种memcached命令对它进行操作。基于memcached开发的应用完全不需要做任何修改。

## Memacheq实现消息队列

在处理业务逻辑时有可能遇到高并发问题，例如商城秒杀、微博评论等。如果不做任何措施可能在高瞬间造成服务器瘫痪，如何解决这个问题呢？队列是个不错的选择。队列（Queue）又称先进先出（First In First Out）利用消息队列可以很好地异步处理数据传送和存储，当你向数据库中写入数据就可采取消息队列来异步插入。只要有并发限制的地方基本都可以使用队列来解决。这里先重点介绍一下memcacheq。

    持久化消息队列memcacheq是一个轻量级的消息队列。依附于Berkeley DB和libevent。Berkeley DB用于持久化存储队列的数据，避免在memcacheq出问题时造成数据丧失。

---

## 参考:

[https://blog.csdn.net/tiedanzi/article/details/53035905](https://blog.csdn.net/tiedanzi/article/details/53035905)


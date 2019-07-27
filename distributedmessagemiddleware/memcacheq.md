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

1. 队列数据存储与BDB，持久保存。

1. 并发性能好。

1. 支持多条队列。

Memcacheq依赖于Brekeley DB和libevent。Berkeley DB 用于持久化存储队列的数据，避免在memcacheq崩溃或这服务器当掉时候，不至于数据丢失。对于并发量较高的web环境，特别是数据库写入操作过多的情景，使用队列可大大缓解因并发问题造成的数据库锁死问题。

Memcacheq的最大优势是：它是基于memcached开发的，可以通过各种memcached命令对它进行操作。基于memcached开发的应用完全不需要做任何修改。

---

作者：张tiedan

来源：CSDN

原文：[https://blog.csdn.net/tiedanzi/article/details/53035905](https://blog.csdn.net/tiedanzi/article/details/53035905)

版权声明：本文为博主原创文章，转载请附上博文链接！





## 参考:

[https://blog.csdn.net/tiedanzi/article/details/53035905](https://blog.csdn.net/tiedanzi/article/details/53035905)




## memcached和memcacheq有什么关系和联系，区别？

#### memcacheq:用于作队列

memcacheq依赖于libevent和BerkleyDB

BerkleyDB用于持久化存储队列的数据。 这样在MEMCACHEQ崩溃或者服务器挂掉的时候，

不至于造成数据的丢失

#### memcached:现在主要用于作缓存



## 什么是memcacheq?

Memcacheq是一个基于memcacheq协议、BDB持久数据存储、高性能轻量级队列服务程序。q是queue的意思，是一个列队。




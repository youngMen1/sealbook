## memcached和memcacheq有什么关系和联系，区别？

#### memcached:现在主要用于作缓存

#### memcacheq:用于作队列

memcacheq依赖于libevent和BerkleyDB

BerkleyDB用于持久化存储队列的数据。 这样在MEMCACHEQ崩溃或者服务器挂掉的时候，

不至于造成数据的丢失


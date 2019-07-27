memcache是主流的缓存框架，现在一般不会再用memcache了，redis逐渐代替了memcache。相比memcache，redis有更丰富的数据类型、支持持久化等等优点。

#### **一、内存分配**

常见的内存分配算法有tcmalloc、jemalloc、malloc等。memcache使用的是jemalloc，除了memcache之外，netty和redis也是使用jemalloc。jemalloc算法主要思路是，将一块大的内存分为多个不同大小的小块的内存，当需要申请内存时，找到最匹配的内存块。

memcache启动时，会先申请一块内存（默认64M），然后计算初始chunk大小，以及slab的个数。初始chunk大小=48B+item\_size+对齐字节，其中48B可以用启动参数-n配置，item\_size是memcache对key-value封装之后的对象大小，固定48B，对齐字节保证最后的大小正好是8的整数倍。所以，初始chunk大小默认是96B。slab个数则和初始chunk大小、增长因子、Page大小有关。增长因子默认是1.25，可以用启动参数-f指定；Page大小默认1M，可以用启动参数-I（这是i/j/k的i的大写，不是L的小写，我曾经掉到这个坑里）指定。slab的个数为：96B、120B、152B...以增长因子（1.25倍）递增直到Page大小（1M），共42个。

当有数据需要存储时，根据大小找到对应的slab，查看是否有空闲的chunk，如果没有，就去申请一个Page，然后将Page分割为一组chunk，用来存储。

一种slab可以申请多个Page，每个Page一旦分配就不会被回收或者重新分配。slab下闲置的chunk不会被其他的slab使用。

#### 二、两级哈希

在客户端会先通过一致性哈希找到key应该被存储到哪个memcache服务器，发送网络请求到对应的服务器之后，服务器是用数组+链表的方式来存储key-value的，所以需要计算key应该存到数组的哪个位置（用key来hash，然后取余）。也就是说，memcache服务器用哈希表来存储所有的key-value。

memcache服务器不向redis，前者是多线程处理客户端请求的，而后者是单线程的。多线程处理请求，就会导致并发操作数组，memcache用类似JDK7中的ConcurrentHashMap的分段锁来保证线程安全。随着hash扩容，锁的数量不会变多，而是每个锁锁住的元素增多。

当存入的key-value达到一定条件时（元素个数超过数组长度的1.5倍），会触发hash扩容，扩容时先申请一个2倍的数组，然后就是迁移数据了。迁移数据不是一次全部迁移完成，而是每次迁移一定数量的桶位（默认1个，可配置），多次迁移之后完成扩容。当要操作一个key时，会比较当前迁移到的桶位下标，如果小于，就去操作原数组，否则操作新数组。

  


  


作者：mrchen004

  


链接：https://www.jianshu.com/p/ec7ebe759c44

  


来源：简书

  


简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

## 参考:

[https://blog.csdn.net/luotuo44?t=1](https://blog.csdn.net/luotuo44?t=1)

[https://www.jianshu.com/p/ec7ebe759c44](https://www.jianshu.com/p/ec7ebe759c44)


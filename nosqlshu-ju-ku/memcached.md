memcache是主流的缓存框架，现在一般不会再用memcache了，redis逐渐代替了memcache。相比memcache，redis有更丰富的数据类型、支持持久化等等优点。

#### **一、内存分配**

常见的内存分配算法有tcmalloc、jemalloc、malloc等。memcache使用的是jemalloc，除了memcache之外，netty和redis也是使用jemalloc。jemalloc算法主要思路是，将一块大的内存分为多个不同大小的小块的内存，当需要申请内存时，找到最匹配的内存块。

memcache启动时，会先申请一块内存（默认64M），然后计算初始chunk大小，以及slab的个数。初始chunk大小=48B+item\_size+对齐字节，其中48B可以用启动参数-n配置，item\_size是memcache对key-value封装之后的对象大小，固定48B，对齐字节保证最后的大小正好是8的整数倍。所以，初始chunk大小默认是96B。slab个数则和初始chunk大小、增长因子、Page大小有关。增长因子默认是1.25，可以用启动参数-f指定；Page大小默认1M，可以用启动参数-I（这是i/j/k的i的大写，不是L的小写，我曾经掉到这个坑里）指定。slab的个数为：96B、120B、152B...以增长因子（1.25倍）递增直到Page大小（1M），共42个。

当有数据需要存储时，根据大小找到对应的slab，查看是否有空闲的chunk，如果没有，就去申请一个Page，然后将Page分割为一组chunk，用来存储。

一种slab可以申请多个Page，每个Page一旦分配就不会被回收或者重新分配。slab下闲置的chunk不会被其他的slab使用。

#### 二、两级哈希

## 参考:

[https://blog.csdn.net/luotuo44?t=1](https://blog.csdn.net/luotuo44?t=1)

[https://www.jianshu.com/p/ec7ebe759c44](https://www.jianshu.com/p/ec7ebe759c44)


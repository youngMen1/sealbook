# 开篇词 | 秒杀系统架构设计都有哪些关键点

TODO 

# 总结



### 秒杀其实主要解决两个问题，一个是并发读，一个是并发写。

### 其实，秒杀的整体架构可以概括为“稳、准、快”几个关键字。

* 高性能。 秒杀涉及大量的并发读和并发写，因此支持高并发访问这点非常关键。本专栏将从设计数据的动静分离方案、热点的发现与隔离、请求的削峰与分层过滤、服务端的极致优化这 4 个方面重点介绍。

* 一致性。 秒杀中商品减库存的实现方式同样关键。可想而知，有限数量的商品在同一时刻被很多倍的请求同时来减库存，减库存又分为“拍下减库存”“付款减库存”以及预扣等几种，在大并发更新的过程中都要保证数据的准确性，其难度可想而知。因此，我将用一篇文章来专门讲解如何设计秒杀减库存方案。

* 高可用。 虽然我介绍了很多极致的优化思路，但现实中总难免出现一些我们考虑不到的情况，所以要保证系统的高可用和正确性，我们还要设计一个 PlanB 来兜底，以便在最坏情况发生时仍然能够从容应对。专栏的最后，我将带你思考可以从哪些环节来设计兜底方案。

### 架构师的角度来看
一个架构师的角度来看，要想打造并维护一个超大流量并发读写、高性能、高可用的系统，在整个用户请求路径上从浏览器到服务端我们要遵循几个原则，就是要保证用户请求的数据尽量少、请求数尽量少、路径尽量短、依赖尽量少，并且不要有单点。这些关键点我会在后面的文章里重点讲解。
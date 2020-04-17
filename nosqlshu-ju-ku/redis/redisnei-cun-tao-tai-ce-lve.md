# Redis内存淘汰策略

现在实际项目中用到redis的越来越多，今天心血来潮研究了下Redis内存的淘汰策略。

所谓内存淘汰策略，就是在往redis里面存储key时内存不足执行的策略。

首先使用Docker启动redis

```
docker run -d -v /usr/local/etc/redis/redis.conf:/usr/local/etc/redis/redis.conf -p 6379:6379 --name redis redis redis-server /usr/local/etc/redis/redis.conf
```

需要在redis官网下载获取最新的redis.conf放在/usr/local/etc/redis/

总共有8种策略，以下是redis.conf关于内存淘汰策略的原文介绍

```
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
# volatile-lru -> Evict using approximated LRU among the keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key among the ones with an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
#
# LRU means Least Recently Used
# LFU means Least Frequently Used
#
# Both LRU, LFU and volatile-ttl are implemented using approximated
# randomized algorithms.
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy noeviction
```

下面对各种策略进行测试

将redis最大内存调整为1m，修改redis.conf

```
maxmemory 1048576
```

修改内存淘汰策略

```
maxmemory-policy noeviction

```




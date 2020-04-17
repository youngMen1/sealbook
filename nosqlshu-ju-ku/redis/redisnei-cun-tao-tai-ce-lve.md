# Redis内存淘汰策略

现在实际项目中用到redis的越来越多，今天心血来潮研究了下Redis内存的淘汰策略。

所谓内存淘汰策略，就是在往redis里面存储key时内存不足执行的策略。

首先使用Docker启动redis

```
docker run -d -v /usr/local/etc/redis/redis.conf:/usr/local/etc/redis/redis.conf -p 6379:6379 --name redis redis redis-server /usr/local/etc/redis/redis.conf
```

需要在redis官网下载获取最新的redis.conf放在/usr/local/etc/redis/

总共有8种策略，以下是redis.conf关于内存淘汰策略的原文介绍


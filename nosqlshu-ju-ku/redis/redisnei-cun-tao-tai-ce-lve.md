Redis内存淘汰策略

现在实际项目中用到redis的越来越多，今天心血来潮研究了下Redis内存的淘汰策略。

所谓内存淘汰策略，就是在往redis里面存储key时内存不足执行的策略。

首先使用Docker启动redis

  





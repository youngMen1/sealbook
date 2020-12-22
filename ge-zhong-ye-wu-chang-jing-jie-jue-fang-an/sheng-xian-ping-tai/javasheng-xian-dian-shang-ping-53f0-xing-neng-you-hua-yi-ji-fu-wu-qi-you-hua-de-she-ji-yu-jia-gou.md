# Java生鲜电商平台-性能优化以及服务器优化的设计与架构

说明：Java开源生鲜电商平台-性能优化以及服务器优化的设计与架构，我采用以下三种维度来讲解
1.代码层面。

2.数据库层面。

3.服务器层面

诚然，性能优化这个方面的确是一个长期的过程，并不是大伙们看了我的文章后就觉得可以做的很好的，我这边只是起一个抛砖引玉的作用，给大伙一种解决问题的思路与方向。

1.Java代码层面优化
补充说明：Java代码层面优化，你需要知道的是，那些代码需要优化，我们知道八二定律告诉我们，80%的性能问题出在20%的代码上，我们需要的是找到那些20%的代码进行针

对性的优化，这样才能把服务的质量优化得最好。


```
那么如何进行监控呢？又怎么样进行监控呢？可以先看前一篇文章：<Java开源生鲜电商平台-监控模块的设计与架构(源码可下载）>.  具体情况大家可以去看。
```

我列举几个常见的优化方案：用实战来说明一切。

看以下代码：

我们知道，买家在支付成功后，支付宝或者微信会服务端回调，然后我们处理自己的业务。

相应的业务，我们在订单服务器层创建一个方法：



```
/**
     * 支付返回
     * @return orderInfo 订单信息
     */
    public void payReturn(OrderInfo orderInfo,PayLogs payLogs);
```

业务实现有以下需要注意的，（我只分享核心内容，非核心的大家自己去看源代码即可）

1.更新支付状态，变成已付款，

2.更新配送状态，待配送。

3.更改交易日志表，变成已经付款。

4.更新订单明细表，记录所有的订单明细都有效。

5.更新用户的余额为0,

6.记录相关的操作日志等等。

相应的代码如下：（spring 事物控制在服务层，如果以上6个步骤有一个错误，则全部回滚。）

```
@Override
    public void payReturn(OrderInfo orderInfo, PayLogs payLogs) {
        orderInfoDao.payReturn(orderInfo);
        orderItemDao.updateOrderItemByOrderNumber(orderInfo.getOrderNumber());
        buyerDao.updateBuyerBalanceToZero(orderInfo.getBuyerId());
        payLogsDao.updatePayLogs(payLogs);        logDao.insertOperatorLogs(orderInfo,payLogs);
    }
```

我们发现，以上6补需要采用5个数据库连接才可以完成，而且在同一个事务里面，因为需要保证数据的一致性。

我们发现，整个业务的操作需要2s完成，对于我们监控业务在500ms的目标，相距太远了，怎么办？

以上代码，究竟那块的性能最差呢？

orderInfoDao.payReturn(orderInfo);      花费：100ms

orderItemDao.updateOrderItemByOrderNumber(orderInfo.getOrderNumber());  花费300ms

buyerDao.updateBuyerBalanceToZero(orderInfo.getBuyerId());   花费200ms

payLogsDao.updatePayLogs(payLogs);  花费800ms

logDao.insertOperatorLogs(orderInfo,payLogs); 花费600ms

综合以上分析，我们需要把同步改成异步，分析出问题的关键。

payLogsDao.updatePayLogs(payLogs); 这块代码进行了优化。

我们惊奇的发现，用户存在刷单的情况，就是不停的支付，就是不支付，对于线上1000多个买家而言，平均每天2单-5单，每单平均订单数在1-10之间

那么一个月的订单日志记录就是：1000*30*5*10=1500000记录，我的天，问题出在这里。日志表也有巨大的量。

最终解决方案：同步改异步进行处理。用redis队列处理，最终性能提高到了500ms内。

一个核心的思想就是：找出问题，解决问题，分而治之的原理进行处理。但是大部分人都输在找问题在，不是找问题难，而是找到核心出问题的代码难。监控那篇文章大伙可以再看看。

2.数据库方面

补充说明：数据库方面无外乎就是关联查询的时候，增加索引，使查询性能更高。那么到底如何做呢？

查询优化:
1.使用慢查询日志去发现慢查询。 
2.使用执行计划去判断查询是否正常运行。 
3.总是去测试你的查询看看是否他们运行在最佳状态下 –久而久之性能总会变化。 
4.避免在整个表上使用count(*),它可能锁住整张表。

相关的优化理论，我已经整理到以前的一篇文章了，这边就不再列举

查看《Mysql性能优化》---https://www.cnblogs.com/jurendage/p/3798703.html

3.服务器监控

说明:我们所说的服务器优化，很大部分是指的tomcat的优化，而不是大家所说的Linux本身的优化，当然这样文章很多，笔者只是用实际说话，看看我们的B2B电商平台如何进行服务器性能的优化。

3.1.tomcat的JVM优化。

这个根据大家自己的电脑进行配置，具体情况需要大家百度自己研究，说来话长



```
#!/bin/sh
JAVA_OPTS="-server -Xms1024M -Xmx1024M -Xss512k -XX:+AggressiveOpts -XX:+UseBiasedLocking -XX:+DisableExplicitGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/log/buyer/gcdump -XX:MaxTenuringThreshold=15 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -Djava.awt.headless=true"
```
这个是笔者的阿里云的服务器的优化。

3.2.tomcat的连接池优化


```
<Connector port="8081" protocol="org.apache.coyote.http11.Http11NioProtocol"
                           connectionTimeout="20000"
                           maxConnections="1000"
                           maxThreads="100"
                           minSpareThreads="50"
                           acceptCount="500"
                           enableLookups="false"
                           compression="on"
                           URIEncoding="UTF-8"
                           redirectPort="8443" />
```



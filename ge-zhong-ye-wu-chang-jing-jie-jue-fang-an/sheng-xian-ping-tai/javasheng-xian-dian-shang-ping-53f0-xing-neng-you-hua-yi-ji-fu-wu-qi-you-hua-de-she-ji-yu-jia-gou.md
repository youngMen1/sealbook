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



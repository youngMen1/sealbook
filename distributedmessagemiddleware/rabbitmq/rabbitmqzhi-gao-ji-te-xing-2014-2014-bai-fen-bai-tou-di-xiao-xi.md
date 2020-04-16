# 1.基本介绍

## 消息发送确认，消费确认整体流程图

![img](/static/image/RabbitMq消息确认图.jpg)

### 什么是生产端的可靠性投递？

保障消息的成功发出

保障MQ节点的成功接收

发送端收到MQ节点\(Broker\) 确认应答

完善的消息补偿机制

如果想保障消息百分百投递成功，只做到前三步不一定能够保障。有些时候或者说有些极端情况，比如生产端在投递消息时可能就失败了，或者说生产端投递了消息，MQ也收到了，MQ在返回确认应答时，由于网络闪断导致生产端没有收到应答，此时这条消息就不知道投递成功了还是失败了，所以针对这些情况我们需要做一些补偿机制。

### 互联网大厂的解决方案

1.消息落库，对消息状态进行打标

2.消息的延迟投递，做二次确认，回调检查 具体使用哪种要根据业务场景和并发量、数据量大小来决定

![img](/static/image/20190618174348115.png)

如上图：

（1）订单服务投递消息给MQ中间件

（2）物流服务监听MQ中间件消息，从而进行消费

我们这篇文章讨论一下，如何保障订单服务把消息成功投递给MQ中间件，以RabbitMQ举例。

## rabbitmq的高级特性：

如何保障消息的百分之百成功？

要满足4个条件：生产方发送出去，消费方接受到消息，发送方接收到消费者的确认信息，完善的消费补偿机制

### 解决方案一、消息落库，进行消息状态打标 ![img](/static/image/1305004-20180908100016978-1607725171.jpg)

该解决方案需要对对数据库进行两次io操作，如果数据量很大，将会导致瓶颈的发生，

本流程是首先将业务入库，发送消息前，将发送的消息存入数据库消息状态可设置为“0“，

两次写入数据库后，发送消息，消息发送后，mq broker 接收并处理消息，完成后发送回馈消息确认信息，

同过确认监听服务器，监听到确认消息后，将其数据库状态更改为“1“，说明消息发送成功。

但是过程中可能出现各种情况导致不能反馈消息，我们需要在添加一个定时轮询任务，

比如说设置最大轮询次数为3，时间间隔为5min，每一次当超过5分钟就去检查一次数据库消息状态，

如果还是“0“则再次发送消息，直到消息状态变为“1“为止。

### 解决方案二、延迟投递，二次确认，回调检查

![img](/static/image/rabbitmq延迟投递.jpg)

该模式适用于高并发的场景，也就是并发量非常大的业务场景，对系统性能要求较高的场合，该模式减少一次主业务的io操作，首先业务落库，然后生成两条消息，首先发出去一次消息，然后5min之后再次发送消息（延迟投递），

第一条发送后，broker端收到后转给消费端，正常处理后，发送一个相应消息（再次生成一条的消息）投递出去step4:send confirm，发送到callback服务中，如果callback服务收确认消息，那么callback将消息写入数据库，5min后发送的第二条消息，进入broker中，此时，broker将消息投递到callback中的监听，收到后检查数据库，

如果检查消息已经存在，说明消费者正常消费，如果，因各种情况发现消费者没有正产消费，那么callback要进行补偿，发起一个RPC通信告诉上游说消息发送失败，上游程序再次生成消息，重新走上述流程。

幂等性：在编程中一个幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。消息重复投递，或者高并发下，消息仅仅消费一次。

# 2.怎么使用

## 介绍

* MailUtil: 发送邮件工具类

* RabbitMqConfig: rabbitmq相关配置

* TestServiceImpl: 生产者, 发送消息

* MailConsumer: 消费者, 消费消息, 发送邮件

* ResendMsgTask: 定时任务, 重新投递发送失败的消息

**涵盖了关于RabbitMQ很多方面的知识点:**

* 消息发送确认机制

* 消费确认机制

* 消息的重新投递

* 消费幂等性, 等等

### 2.1.**163邮箱授权码的获取**

![img](/static/image/微信截图_20200416171330.png)

具体怎么开启百度查

该授权码就是配置文件spring.mail.password需要的密码

### R**abbitmq、邮箱配置**

```
  rabbitmq:
    username: admin
    password: admin@123
    addresses: 192.168.1.28:5672
    listener:
      type: simple
      simple:
        concurrency: 5
        max-concurrency: 20
        acknowledge-mode: manual #设置手动确认
        retry:
          enabled: true
          max-attempts: 3
          initial-interval: 1000ms #尝试时间间隔
        default-requeue-rejected: false #重试失败后是否回队
    connection-timeout: 5000ms
    cache:
      channel:
        size: 5
    publisher-confirms: true #发布者消息确认
    publisher-returns: true  #发布者消息回调
    template:
      retry:
        enabled: true
        max-attempts: 3
        initial-interval: 1000ms #尝试时间间隔
    virtual-host: /

  mail:
    from: fengzhiqiangzxx@163.com
    host: smtp.163.com
    password: fengzhiqiang
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
            required: true
    username: fengzhiqiangzxx@163.com

# 配置数据库
mybatis:
  mapper-locations: classpath:/mapper/*.xml
```

# 3.总结

# 4.参考

自己的源码地址：

Rabbitmq之高级特性——百分百投递消息：  
[https://blog.csdn.net/zuixiaoyao\_001/article/details/82599115](https://blog.csdn.net/zuixiaoyao_001/article/details/82599115)

如何保障订单服务把消息成功投递给MQ中间件，以RabbitMQ举例：

[https://blog.csdn.net/luoyang\_java/article/details/92797134](https://blog.csdn.net/luoyang_java/article/details/92797134)

SpringBoot+RabbitMQ 保证消息100%投递成功实例：

[https://blog.csdn.net/weixin\_44460333/article/details/103942627](https://blog.csdn.net/weixin_44460333/article/details/103942627)

SpringBoot+RabbitMQ ，保证消息100%投递成功并被消费（附源码）:

[https://blog.csdn.net/weixin\_38405253/article/details/103740420?depth\_1-utm\_source=distribute.pc\_relevant.none-task-blog-OPENSEARCH-1&utm\_source=distribute.pc\_relevant.none-task-blog-OPENSEARCH-1](https://blog.csdn.net/weixin_38405253/article/details/103740420?depth_1-utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-1&utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-1)


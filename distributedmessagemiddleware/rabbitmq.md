## 五种队列模式

![](/assets/20180805224706123.png)

## 描述:

RabbitMQ是使用Erlang语言开发的开源消息队列系统，基于AMQP\(AMQP（二进制），STOMP（文本），MQTT（二进制）HTTP（里面包装其他协议）等协议。\)协议来实现。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。AMQP协议更多用在企业系统内，对数据一致性、稳定性和可靠性要求很高的场景，对性能和吞吐量的要求还在其次。

## 简单队列

一个生产者一个消费者

## work模式\(多对多使用\)

一个生产者多个消费者

## Topic Exchange\(主题模式\)

## Fanout Exchange\(订阅模式\)

[5.学习五种队列](https://blog.csdn.net/hellozpc/article/details/81436980#5_192)

* * [5.1.导入my-rabbitmq项目](https://blog.csdn.net/hellozpc/article/details/81436980#51myrabbitmq_194)
  * [5.2.简单队列](https://blog.csdn.net/hellozpc/article/details/81436980#52_198)
  * [5.3.Work模式](https://blog.csdn.net/hellozpc/article/details/81436980#53Work_323)
  * [5.4.Work模式的“能者多劳”](https://blog.csdn.net/hellozpc/article/details/81436980#54Work_478)
  * [5.5.消息的确认模式](https://blog.csdn.net/hellozpc/article/details/81436980#55_500)
  * [5.6.订阅模式](https://blog.csdn.net/hellozpc/article/details/81436980#56_515)
  * [5.7.路由模式](https://blog.csdn.net/hellozpc/article/details/81436980#57_673)
  * [5.8.主题模式（通配符模式）](https://blog.csdn.net/hellozpc/article/details/81436980#58_684)

## 二个概念

轮询分发 ：使用任务队列的优点之一就是可以轻易的并行工作。如果我们积压了好多工作，我们可以通过增加工作者（消费者）来解决这一问题，使得系统的伸缩性更加容易。在默认情况下，RabbitMQ将逐个发送消息到在序列中的下一个消费者\(而不考虑每个任务的时长等等，且是提前一次性分配，并非一个一个分配\)。平均每个消费者获得相同数量的消息。这种方式分发消息机制称为Round-Robin（轮询）。

公平分发 ：虽然上面的分配法方式也还行，但是有个问题就是：比如：现在有2个消费者，所有的奇数的消息都是繁忙的，而偶数则是轻松的。按照轮询的方式，奇数的任务交给了第一个消费者，所以一直在忙个不停。偶数的任务交给另一个消费者，则立即完成任务，然后闲得不行。而RabbitMQ则是不了解这些的。这是因为当消息进入队列，RabbitMQ就会分派消息。它不看消费者为应答的数目，只是盲目的将消息发给轮询指定的消费者。

### 确认模式:

* basic.ack     用于肯定确认
* basic.nack   用于否定确认（注意：这是[AMQP 0-9-1](https://www.rabbitmq.com/nack.html)的[RabbitMQ扩展](https://www.rabbitmq.com/nack.html)）
* basic.reject 用于否定确认，但与basic.nack相比有一个限制

消息的确认模式

消费者从队列中获取消息，服务端如何知道消息已经被消费呢？

模式1：自动确认

只要消息从队列中获取，无论消费者获取到消息后是否成功消息，都认为是消息已经成功消费。

```
// 监听队列，false表示手动返回完成状态，true表示自动
channel.basicConsume(QUEUE_NAME, false, consumer);
```

模式2：手动确认

消费者从队列中获取消息后，服务器会将该消息标记为不可用状态，等待消费者的反馈，如果消费者一直没有反馈，那么该消息将一直处于不可用状态。

```
// 返回确认状态，表示使用手动确认模式
 channel.basicAck(tag, false);
```

反馈消息的消费状态

```
// 反馈消息的消费状态
channel.basicNack(tag, false, false);
```

## 注意:

#### 发送不起作用:

如果这是您第一次使用RabbitMQ并且没有看到“已发送”消息，那么您可能会想到可能出现的问题。也许代理是在没有足够的可用磁盘空间的情况下启动的（默认情况下它至少需要200 MB空闲），因此拒绝接受消息。检查代理日志文件以确认并在必要时减少限制。该[配置文件文档](https://www.rabbitmq.com/configure.html#config-items)会告诉你如何设置disk\_free\_limit。

参考:

[https://www.rabbitmq.com/tutorials/tutorial-one-python.html](https://www.rabbitmq.com/tutorials/tutorial-one-python.html)


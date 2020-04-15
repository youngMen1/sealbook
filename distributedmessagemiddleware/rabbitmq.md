## 五种队列模式

基本组成部分：发送者，交换器，队列，消费者

RabbitMQ提供了6种消息模型，但是_第6种其实是RPC，并不是MQ_，因此不予学习。

* 基本消息模型：生产者–&gt;队列–&gt;消费者

* work消息模型：生产者–&gt;队列–&gt;多个消费者共同消费

* 订阅模型-Fanout：广播，将消息交给所有绑定到交换机的队列，每个消费者都会收到同一条消息

* 订阅模型-Direct：定向，把消息交给符合指定 rotingKey 的队列

* 订阅模型-Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列

**但是其实3、4、5这三种都属于订阅模型，只不过进行路由的方式不同。**

![](/assets/20180805224706123.png)

## 描述:

RabbitMQ是使用Erlang语言开发的开源消息队列系统，基于AMQP\(AMQP（二进制），STOMP（文本），MQTT（二进制）HTTP（里面包装其他协议）等协议。\)协议来实现。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。AMQP协议更多用在企业系统内，对数据一致性、稳定性和可靠性要求很高的场景，对性能和吞吐量的要求还在其次。

## 简单模式

![](/assets/微信截图_20190720163412.png)

1.生产者将消息交给默认的交换机\(AMQP default\)

2.交换机获取消息后交给绑定这个生产者的队列\(这个关系是通过队列名称完成\)

3.监听当前队列的消费者获取消息，执行消费逻辑

应用场景:

* 短信
* 聊天

## 工作模式\(work模式\)

![](/assets/微信截图_20190720163407.png)

资源争抢

当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。此时就可以使用work 模型：

**让多个消费者绑定到一个队列，共同消费队列中的消息，**队列中的消息一旦消费，就会消失，因此任务是不会被重复执行的。

1.生产者交消息交给交换机

2.交换机交给绑定的队列

3.队列由多个消费者同时监听，只有其中一个能够获取这一条消息，形成了资源的争抢，谁的资源空闲大，争抢到的可能越大。

应用场景:

* 抢红包
* 大型系统的资源调度
* ![](/assets/微信截图_20190720163402.png)二个概念  
  轮询分发 ：使用任务队列的优点之一就是可以轻易的并行工作。如果我们积压了好多工作，我们可以通过增加工作者（消费者）来解决这一问题，使得系统的伸缩性更加容易。在默认情况下，RabbitMQ将逐个发送消息到在序列中的下一个消费者\(而不考虑每个任务的时长等等，且是提前一次性分配，并非一个一个分配\)。平均每个消费者获得相同数量的消息。这种方式分发消息机制称为Round-Robin（轮询）。

  公平分发 ：虽然上面的分配法方式也还行，但是有个问题就是：比如：现在有2个消费者，所有的奇数的消息都是繁忙的，而偶数则是轻松的。按照轮询的方式，奇数的任务交给了第一个消费者，所以一直在忙个不停。偶数的任务交给另一个消费者，则立即完成任务，然后闲得不行。而RabbitMQ则是不了解这些的。这是因为当消息进入队列，RabbitMQ就会分派消息。它不看消费者为应答的数目，只是盲目的将消息发给轮询指定的消费者。

## 发布订阅模式\(publish/fanout\)

![](/assets/微信截图_20190720163356.png)

在Fanout模型下:

Fanout，也称为广播。在广播模式下，消息发送流程是这样的：

1） 可以有多个消费者

2） 每个消费者有自己的queue（队列）

3） 每个队列都要绑定到Exchange（交换机）

4） 生产者发送的消息，只能发送到交换机，交换机来决定要发给哪个队列，生产者无法决定。

5） 交换机把消息发送给绑定过的所有队列

6） 队列的消费者都能拿到消息。实现一条消息被多个消费者消费

注意:

在Fanout模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。

1.生产者扔给交换机消息

2.交换机根据自身的类型\(fanout\)将会把所有消息复杂同步到所有与其绑定的队列

3.每个队列可以有一个消费者，接收消息进行消费逻辑

应用场景:

* 邮件群发
* 广告

## 路由模式\(routing/direct\)

![](/assets/微信截图_20190720163346.png)

在Direct模型下：

队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）

消息的发送方在 向 Exchange发送消息时，也必须指定消息的 RoutingKey。

Exchange不再把消息交给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的Routingkey与消息的 Routing key完全一致，才会接收到消息

1.生产者还是将消息发送给交换机，消息携带具体的路由key\(routingKey\)

2.交换机类型direct，将接收到的消息中的routingkey，比对与之绑定的队列的routingKey

3.消费者监听一个队列。获取消息，执行消费逻辑

应用场景:

* 根据生产者的要求发送给特定的一个或者一批队列，错误的通报

## 主题模式\(topic\)

![](/assets/微信截图_20190720163324.png)

在Topic模型下:

同一个消息被多个消费者获取。一个消费者队列可以有多个消费者实例，只有其中一个消费者实例会消费到消息。

topic 是RabbitMQ中最灵活的一种方式，可以根据routing\_key自由的绑定不同的队列

Topic类型的Exchange与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型Exchange可以让队列在绑定Routing key 的时候使用通配符！

Routingkey 一般都是有一个或多个单词组成，多个单词之间以”.”分割，例如： user.insert

通配符规则    举例:

```
#：匹配一个或多个词    user.#：能够匹配user.insert.save 或者 user.insert
*：匹配不多不少恰好1个词    user.*：只能匹配user.insert
```

1.生产者发送消息，消息携带具体的路由key

2.交换机的类型topic

3.队列绑定交换机不在使用具体的路由key而是一个范围值

4.和路由模式的区别:

路由模式中的queue绑定携带的是具体的key值，路由细化划分topic主体模式queue携带的是范围的匹配，某一类的消息获取

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
autoAck：是否自动ack，如果不自动ack，需要使用channel.ack、channel.nack、channel.basicReject 进行消息应答
```

模式2：手动确认

消费者从队列中获取消息后，服务器会将该消息标记为不可用状态，等待消费者的反馈，如果消费者一直没有反馈，那么该消息将一直处于不可用状态。

```
// 返回确认状态，表示使用手动确认模式
 channel.basicAck(tag, false);
```

**channel.basicNack\(delivery.getEnvelope\(\).getDeliveryTag\(\), false, true\);**

```
channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, true);
deliveryTag:该消息的index
multiple：是否批量.true:将一次性拒绝所有小于deliveryTag的消息。
requeue：被拒绝的是否重新入队列
```

## 注意:

#### 发送不起作用:

如果这是您第一次使用RabbitMQ并且没有看到“已发送”消息，那么您可能会想到可能出现的问题。也许代理是在没有足够的可用磁盘空间的情况下启动的（默认情况下它至少需要200 MB空闲），因此拒绝接受消息。检查代理日志文件以确认并在必要时减少限制。该[配置文件文档](https://www.rabbitmq.com/configure.html#config-items)会告诉你如何设置disk\_free\_limit。

---

## 参考:

[https://www.rabbitmq.com/tutorials/tutorial-one-python.html](https://www.rabbitmq.com/tutorials/tutorial-one-python.html)

[https://www.cnblogs.com/Jeely/p/10784172.html](https://www.cnblogs.com/Jeely/p/10784172.html)

[https://blog.csdn.net/qq\_38762237/article/details/89433506](https://blog.csdn.net/qq_38762237/article/details/89433506)

[https://blog.csdn.net/weixin\_42236165/article/details/92795733](https://blog.csdn.net/weixin_42236165/article/details/92795733)


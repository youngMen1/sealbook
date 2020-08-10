# 1.RabbitMQ高级特性幂等性概念及解决方案

## 1.1.幂等性是什么？

简单来说就是用户对于同一操作发起的一次请求或者多次请求的结果是一致的。  
我们可以借鉴数据库的乐观锁机制来举个例子

* 首先为表添加一个版本字段version
* 在执行更新操作前呢，会先去数据库查询这个version
* 然后执行更新语句，以version作为条件，例如：
  ```
  UPDATE T_REPS SET COUNT = COUNT -1，VERSION = VERSION + 1 WHERE VERSION = 1
  ```
* 如果执行更新时有其他人先更新了这张表的数据，那么这个条件就不生效了，也就不会执行操作了，通过这种乐观锁的机制来保障幂等性。

## 1.2.消费端-幂等性保障

### 1.2.1.什么情况下会出现重复消费？

当消费者消费完消息时，在给生产端返回ack时由于网络中断，导致生产端未收到确认信息，该条消息会重新发送并被消费者消费，但实际上该消费者已成功消费了该条消息，这就是重复消费问题。

### 1.2.2.如何避免消息的重复消费问题？

消费端实现幂等性，就意味着，我们的消息永远不会消费多次，即使我们收到了多条一样的消息。

**业界主流的幂等性操作：**

##### 1.唯一ID + 指纹码机制，利用数据库主键去重

##### 2.利用Redis的原子性去实现

#### 1.2.2.1.唯一ID + 指纹码机制，利用数据库主键去重

* 唯一ID + 指纹码机制，利用数据库主键去重
* SELECT COUNT\(1\) FROM T\_ORDER WHERE ID = 唯一ID +指纹码
* 好处：实现简单
* 坏处：高并发下有数据库写入的性能瓶颈
* 解决方案：跟进ID进行分库分表进行算法路由

整个思路就是首先我们需要根据消息生成一个全局唯一的ID，然后还需要加上一个指纹码。这个指纹码它并不一定是系统去生成的，而是一些外部的规则或者内部的业务规则去拼接，它的目的就是为了保障这次操作是绝对唯一的。

将ID + 指纹码拼接好的值作为数据库主键，就可以进行去重了。即在消费消息前呢，先去数据库查询这条消息的指纹码标识是否存在，没有就执行insert操作，如果有就代表已经被消费了，就不需要管了。

对于高并发下的数据库性能瓶颈，可以跟进ID进行分库分表策略，采用一些路由算法去进行分压分流。应该保证ID通过这种算法，消息即使投递多次都落到同一个数据库分片上，这样就由单台数据库幂等变成多库的幂等。

#### 1.2.2.2.利用Redis的原子性去实现

我们都知道redis是单线程的，并且性能也非常好，提供了很多原子性的命令。比如可以使用 `setnx` 命令。

* 通过setnx等命令

```
SET 订单号 时间戳 过期时间
例如:
SET 1893505609317740 1466849127 EX 300 NX
```

在接收到消息后将消息ID作为key执行 setnx 命令，如果执行成功就表示没有处理过这条消息，可以进行消费了，执行失败表示消息已经被消费了。

使用 redis 的原子性去实现主要需要考虑两个点：

* 第一：我们是否要进行数据落库，如果落库的话，关键解决的问题是数据库和缓存如何做到数据一致性（原子性）？
* 第二：如果不进行落库，那么都存储到缓存中，如何设置定时同步的策略\(同步到关系型数据库\)？缓存又如何做到数据可靠性保障呢

关于不落库，定时同步的策略，目前主流方案有两种，第一种为双缓存模式，异步写入到缓存中，也可以异步写到数据库，但是最终会有一个回调函数检查，这样能保障最终一致性，不能保证100%的实时性。第二种是定时同步，比如databus同步。

# 2.RabbitMQ解决消息幂等性问题

关于MQ消费者的幂等性问题，在于MQ的重试机制，因为网络原因或客户端延迟消费导致重复消费。使用MQ重试机制需要注意的事项以及如何解决消费者幂等性问题以下将逐一讲解。

## 1. RabbitMQ自动重试机制

消费者在消费消息的时候，如果消费者业务逻辑出现程序异常，这个时候我们如何处理？

使用`重试机制`，RabbitMQ默认开启重试机制。

**实现原理：**

@RabbitHandler注解 底层使用Aop拦截，如果程序\(消费者\)没有抛出异常，自动提交事务  
如果Aop使用异常通知拦截获取到异常后，自动实现补偿机制，消息缓存在RabbitMQ服务器端

**注意：**

默认会一直重试到消费者不抛异常为止，这样显然不好。我们需要修改重试机制策略，如间隔3s重试一次\)

配置：

```
spring:
  rabbitmq:
    # 连接地址
    host: 127.0.0.1
    # 端口号
    port: 5672
    # 账号
    username: guest
    # 密码
    password: guest
    # 地址(类似于数据库的概念)
    virtual-host: /admin_vhost
    # 消费者监听相关配置
    listener:
      simple:
        retry:
          # 开启消费者(程序出现异常)重试机制，默认开启并一直重试
          enabled: true
          # 最大重试次数
          max-attempts: 5
          # 重试间隔时间(毫秒)
          initial-interval: 3000
```

## 2.如何合理选择重试机制？

情况1: 消费者获取到消息后，调用第三方接口，但接口暂时无法访问，是否需要重试? **需要重试，可能是因为网络原因短暂不能访问**

情况2: 消费者获取到消息后，抛出数据转换异常，是否需要重试? **不需要重试,因为属于程序bug需要重新发布版本**

总结：对于情况2，如果消费者代码抛出异常是需要发布新版本才能解决的问题，那么不需要重试，重试也无济于事。**应该采用日志记录+定时任务job进行健康检查+人工进行补偿      
**

## 3.调用第三方接口自动实现补偿机制

我们知道了，RabbitMQ在消费者消费发生异常时，会自动进行补偿机制，所以我们（消费者）在调用第三方接口时，可以根据返回结果判断是否成功：

* 成功：正常消费
* 失败：手动抛处一个异常，这时RabbitMQ自动给我们做重试 \(补偿\)。

## 4.如何解决消费者幂等性问题，防止重复消费 \(**MQ重试机制需要注意的问题**\)

产生原因:网络延迟传输中，消费者出现异常或者消费者延迟消费，会造成进行MQ重试补偿，在重试过程中，可能会造成重复消费。

面试题：MQ中消费者如何保证幂等性问题，不被重复消费？  
![](/static/image/2019061023554173.png)

伪代码：  
生产者核心代码:

请求头设置消息id（messageId）

```
@Component
public class FanoutProducer {
    @Autowired
    private AmqpTemplate amqpTemplate;

    public void send(String queueName) {
        String msg = "my_fanout_msg:" + System.currentTimeMillis();
        //请求头设置消息id（messageId）
        Message message = MessageBuilder.withBody(msg.getBytes()).setContentType(MessageProperties.CONTENT_TYPE_JSON)
                .setContentEncoding("utf-8").setMessageId(UUID.randomUUID() + "").build();
        System.out.println(msg + ":" + msg);
        amqpTemplate.convertAndSend(queueName, message);
    }
}
```

消费者核心代码：

```
@RabbitListener(queues = "fanout_email_queue")
    public void process(Message message) throws Exception {
        // 获取消息Id
        String messageId = message.getMessageProperties().getMessageId();
        String msg = new String(message.getBody(), "UTF-8");
        //② 判断唯一Id是否被消费，消息消费成功后将id和状态保存在日志表中，我们从（①步骤）表中获取并判断messageId的状态即可
        //从redis中获取messageId的value
        String value = redisUtils.get(messageId)+"";
        if(value.equals("1") ){ //表示已经消费
            return; //结束
        }
        System.out.println("邮件消费者获取生产者消息" + "messageId:" + messageId + ",消息内容:" + msg);
        JSONObject jsonObject = JSONObject.parseObject(msg);
        // 获取email参数
        String email = jsonObject.getString("email");
        // 请求地址
        String emailUrl = "http://127.0.0.1:8083/sendEmail?email=" + email;
        JSONObject result = HttpClientUtils.httpGet(emailUrl);
        if (result == null) {
            // 因为网络原因,造成无法访问,继续重试
            throw new Exception("调用接口失败!");
        }
        System.out.println("执行结束....");
        //① 执行到这里已经消费成功，我们可以修改messageId的状态，并存入日志表(可以存到redis中，key为消息Id、value为状态)
    }
```

## 5.SpringBoot整合RabbitMQ应答模式\(ACK\)

1.修改配置simple下添加 **acknowledge-mode: manual：**

```
spring:
  rabbitmq:
    # 连接地址
    host: 127.0.0.1
    # 端口号
    port: 5672
    # 账号
    username: guest
    # 密码
    password: guest
    # 地址(类似于数据库的概念)
    virtual-host: /admin_vhost
    # 消费者监听相关配置
    listener:
      simple:
        retry:
          # 开启消费者(程序出现异常)重试机制，默认开启并一直重试
          enabled: true
          # 最大重试次数
          max-attempts: 5
          # 重试间隔时间(毫秒)
          initial-interval: 3000
        # 开启手动ack
        acknowledge-mode: manual
```

2.消费者增加代码:

```
Long deliveryTag = (Long) headers.get(AmqpHeaders.DELIVERY_TAG); 手动ack
channel.basicAck(deliveryTag, false);手动签收
```

```
// 邮件队列
@Component
public class FanoutEamilConsumer {
    @RabbitListener(queues = "fanout_email_queue")
    public void process(Message message, @Headers Map<String, Object> headers, Channel channel) throws Exception {
        System.out
                .println(Thread.currentThread().getName() + ",邮件消费者获取生产者消息msg:" + new String(message.getBody(), "UTF-8")
                        + ",messageId:" + message.getMessageProperties().getMessageId());
        // 手动ack
        Long deliveryTag = (Long) headers.get(AmqpHeaders.DELIVERY_TAG);
        // 手动签收
        channel.basicAck(deliveryTag, false);
    }
}
```

# 3.参考

参考课程:[https://coding.imooc.com/class/262.html](https://links.jianshu.com/go?to=https%3A%2F%2Fcoding.imooc.com%2Fclass%2F262.html)

RabbitMQ解决消息幂等性问题:[https://blog.csdn.net/qq\_38252039/article/details/91409955](https://blog.csdn.net/qq_38252039/article/details/91409955)

RocketMQ消息幂等的通用解决方案：https://mp.weixin.qq.com/s/X25Jw-sz3XItVrXRS6IQdg




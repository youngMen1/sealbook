# RabbitMQ延迟队列

## 1.说明

在上一篇中，介绍了RabbitMQ中的死信队列是什么，何时使用以及如何使用RabbitMQ的死信队列。相信通过上一篇的学习，对于死信队列已经有了更多的了解，这一篇的内容也跟死信队列息息相关，如果你还不了解死信队列，那么建议你先进行上一篇文章的阅读。

这一篇里，我们将继续介绍RabbitMQ的高级特性，通过本篇的学习，你将收获：

1.什么是延时队列  
2.延时队列使用场景  
3.RabbitMQ中的TTL  
4.如何利用RabbitMQ来实现延时队列

## 2.本文大纲

以下是本文大纲：  
![](/static/image/5d3d74d99699d43032.png)  
本文阅读前，需要对RabbitMQ以及死信队列有一个简单的了解。

## 3.什么是延时队列

**延时队列**，首先，它是一种队列，队列意味着内部的元素是**有序**的，元素出队和入队是有方向性的，元素从一端进入，从另一端取出。

其次，**延时队列**，最重要的特性就体现在它的**延时**属性上，跟普通的队列不一样的是，**普通队列中的元素总是等着希望被早点取出处理，而延时队列中的元素则是希望被在指定时间得到取出和处理**，所以延时队列中的元素是都是带时间属性的，通常来说是需要被处理的消息或者任务。

简单来说，延时队列就是用来存放需要在指定时间被处理的元素的队列。

## 4.延时队列使用场景

那么什么时候需要用延时队列呢？考虑一下以下场景：

订单在十分钟之内未支付则自动取消。  
新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。  
账单在一周内未支付，则自动结算。  
用户注册成功后，如果三天内没有登陆则进行短信提醒。  
用户发起退款，如果三天内没有得到处理则通知相关运营人员。\`\`  
预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议。  
这些场景都有一个特点，需要在某个事件发生之后或者之前的指定时间点完成某一项任务，如：发生订单生成事件，在十分钟之后检查该订单支付状态，然后将未支付的订单进行关闭；发生店铺创建事件，十天后检查该店铺上新商品数，然后通知上新数为0的商户；发生账单生成事件，检查账单支付状态，然后自动结算未支付的账单；发生新用户注册事件，三天后检查新注册用户的活动数据，然后通知没有任何活动记录的用户；发生退款事件，在三天之后检查该订单是否已被处理，如仍未被处理，则发送消息给相关运营人员；发生预定会议事件，判断离会议开始是否只有十分钟了，如果是，则通知各个与会人员。

看起来似乎使用定时任务，一直轮询数据，每秒查一次，取出需要被处理的数据，然后处理不就完事了吗？如果数据量比较少，确实可以这样做，比如：对于“如果账单一周内未支付则进行自动结算”这样的需求，如果对于时间不是严格限制，而是宽松意义上的一周，那么每天晚上跑个定时任务检查一下所有未支付的账单，确实也是一个可行的方案。但对于数据量比较大，并且时效性较强的场景，如：“订单十分钟内未支付则关闭“，短期内未支付的订单数据可能会有很多，活动期间甚至会达到百万甚至千万级别，对这么庞大的数据量仍旧使用轮询的方式显然是不可取的，很可能在一秒内无法完成所有订单的检查，同时会给数据库带来很大压力，无法满足业务要求而且性能低下。

更重要的一点是，不！优！雅！

没错，作为一名有追求的程序员，始终应该追求更优雅的架构和更优雅的代码风格，写代码要像写诗一样优美。【滑稽】

这时候，延时队列就可以闪亮登场了，以上场景，正是延时队列的用武之地。

既然**延时队列**可以解决很多特定场景下，带时间属性的任务需求，那么如何构造一个延时队列呢？接下来，本文将介绍如何用RabbitMQ来实现延时队列。

## 5.RabbitMQ中的TTL

在介绍延时队列之前，还需要先介绍一下RabbitMQ中的一个高级特性——**TTL（Time To Live）**。

**TTL**是什么呢？**TTL**是RabbitMQ中一个消息或者队列的属性，表明**一条消息或者该队列中的所有消息的最大存活时间，单位是毫秒。**换句话说，如果一条消息设置了TTL属性或者进入了设置TTL属性的队列，那么这条消息如果在TTL设置的时间内没有被消费，则会成为“死信”（至于什么是死信，请翻看上一篇）。如果同时配置了队列的TTL和消息的TTL，那么较小的那个值将会被使用。

那么，如何设置这个TTL值呢？有两种方式，第一种是在创建队列的时候设置队列的“x-message-ttl”属性，如下：

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 6000);
channel.queueDeclare(queueName, durable, exclusive, autoDelete, args);
```

这样所有被投递到该队列的消息都最多不会存活超过6s。

另一种方式便是针对每条消息设置TTL，代码如下：

```
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
builder.expiration("6000");
AMQP.BasicProperties properties = builder.build();
channel.basicPublish(exchangeName, routingKey, mandatory, properties, "msg body".getBytes());
```

这样这条消息的过期时间也被设置成了6s。

但这两种方式是有区别的，如果设置了队列的TTL属性，那么一旦消息过期，就会被队列丢弃，而第二种方式，消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间。

另外，还需要注意的一点是，如果不设置TTL，表示消息永远不会过期，如果将TTL设置为0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。

## 6.如何利用RabbitMQ实现延时队列

前一篇里介绍了如果设置死信队列，前文中又介绍了TTL，至此，利用RabbitMQ实现延时队列的两大要素已经集齐，接下来只需要将它们进行调和，再加入一点点调味料，延时队列就可以新鲜出炉了。

想想看，**延时队列**，不就是想要消息延迟多久被处理吗，TTL则刚好能让消息在延迟多久之后成为死信，另一方面，成为死信的消息都会被投递到死信队列里，这样只需要消费者一直消费死信队列里的消息就万事大吉了，因为里面的消息都是希望被立即处理的消息。

从下图可以大致看出消息的流向：  
![](/static/image/5d3d743143ecc85643.png)  
生产者生产一条延时消息，根据需要延时时间的不同，利用不同的routingkey将消息路由到不同的延时队列，每个队列都设置了不同的TTL属性，并绑定在同一个死信交换机中，消息过期后，根据routingkey的不同，又会被路由到不同的死信队列中，消费者只需要监听对应的死信队列进行处理即可。

下面来看代码：

先声明交换机、队列以及他们的绑定关系：

```
@Configuration
public class RabbitMQConfig {

    public static final String DELAY_EXCHANGE_NAME = "delay.queue.demo.business.exchange";
    public static final String DELAY_QUEUEA_NAME = "delay.queue.demo.business.queuea";
    public static final String DELAY_QUEUEB_NAME = "delay.queue.demo.business.queueb";
    public static final String DELAY_QUEUEA_ROUTING_KEY = "delay.queue.demo.business.queuea.routingkey";
    public static final String DELAY_QUEUEB_ROUTING_KEY = "delay.queue.demo.business.queueb.routingkey";
    public static final String DEAD_LETTER_EXCHANGE = "delay.queue.demo.deadletter.exchange";
    public static final String DEAD_LETTER_QUEUEA_ROUTING_KEY = "delay.queue.demo.deadletter.delay_10s.routingkey";
    public static final String DEAD_LETTER_QUEUEB_ROUTING_KEY = "delay.queue.demo.deadletter.delay_60s.routingkey";
    public static final String DEAD_LETTER_QUEUEA_NAME = "delay.queue.demo.deadletter.queuea";
    public static final String DEAD_LETTER_QUEUEB_NAME = "delay.queue.demo.deadletter.queueb";

    // 声明延时Exchange
    @Bean("delayExchange")
    public DirectExchange delayExchange(){
        return new DirectExchange(DELAY_EXCHANGE_NAME);
    }

    // 声明死信Exchange
    @Bean("deadLetterExchange")
    public DirectExchange deadLetterExchange(){
        return new DirectExchange(DEAD_LETTER_EXCHANGE);
    }

    // 声明延时队列A 延时10s
    // 并绑定到对应的死信交换机
    @Bean("delayQueueA")
    public Queue delayQueueA(){
        Map<String, Object> args = new HashMap<>(2);
        // x-dead-letter-exchange    这里声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE);
        // x-dead-letter-routing-key  这里声明当前队列的死信路由key
        args.put("x-dead-letter-routing-key", DEAD_LETTER_QUEUEA_ROUTING_KEY);
        // x-message-ttl  声明队列的TTL
        args.put("x-message-ttl", 6000);
        return QueueBuilder.durable(DELAY_QUEUEA_NAME).withArguments(args).build();
    }

    // 声明延时队列B 延时 60s
    // 并绑定到对应的死信交换机
    @Bean("delayQueueB")
    public Queue delayQueueB(){
        Map<String, Object> args = new HashMap<>(2);
        // x-dead-letter-exchange    这里声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE);
        // x-dead-letter-routing-key  这里声明当前队列的死信路由key
        args.put("x-dead-letter-routing-key", DEAD_LETTER_QUEUEB_ROUTING_KEY);
        // x-message-ttl  声明队列的TTL
        args.put("x-message-ttl", 60000);
        return QueueBuilder.durable(DELAY_QUEUEB_NAME).withArguments(args).build();
    }

    // 声明死信队列A 用于接收延时10s处理的消息
    @Bean("deadLetterQueueA")
    public Queue deadLetterQueueA(){
        return new Queue(DEAD_LETTER_QUEUEA_NAME);
    }

    // 声明死信队列B 用于接收延时60s处理的消息
    @Bean("deadLetterQueueB")
    public Queue deadLetterQueueB(){
        return new Queue(DEAD_LETTER_QUEUEB_NAME);
    }

    // 声明延时队列A绑定关系
    @Bean
    public Binding delayBindingA(@Qualifier("delayQueueA") Queue queue,
                                    @Qualifier("delayExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with(DELAY_QUEUEA_ROUTING_KEY);
    }

    // 声明业务队列B绑定关系
    @Bean
    public Binding delayBindingB(@Qualifier("delayQueueB") Queue queue,
                                    @Qualifier("delayExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with(DELAY_QUEUEB_ROUTING_KEY);
    }

    // 声明死信队列A绑定关系
    @Bean
    public Binding deadLetterBindingA(@Qualifier("deadLetterQueueA") Queue queue,
                                    @Qualifier("deadLetterExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with(DEAD_LETTER_QUEUEA_ROUTING_KEY);
    }

    // 声明死信队列B绑定关系
    @Bean
    public Binding deadLetterBindingB(@Qualifier("deadLetterQueueB") Queue queue,
                                      @Qualifier("deadLetterExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with(DEAD_LETTER_QUEUEB_ROUTING_KEY);
    }
}
```

接下来，创建两个消费者，分别对两个死信队列的消息进行消费：

```
@Slf4j
@Component
public class DeadLetterQueueConsumer {

    @RabbitListener(queues = DEAD_LETTER_QUEUEA_NAME)
    public void receiveA(Message message, Channel channel) throws IOException {
        String msg = new String(message.getBody());
        log.info("当前时间：{},死信队列A收到消息：{}", new Date().toString(), msg);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }

    @RabbitListener(queues = DEAD_LETTER_QUEUEB_NAME)
    public void receiveB(Message message, Channel channel) throws IOException {
        String msg = new String(message.getBody());
        log.info("当前时间：{},死信队列B收到消息：{}", new Date().toString(), msg);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }
}
```

然后是消息的生产者：

```
@Component
public class DelayMessageSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendMsg(String msg, DelayTypeEnum type){
        switch (type){
            case DELAY_10s:
                rabbitTemplate.convertAndSend(DELAY_EXCHANGE_NAME, DELAY_QUEUEA_ROUTING_KEY, msg);
                break;
            case DELAY_60s:
                rabbitTemplate.convertAndSend(DELAY_EXCHANGE_NAME, DELAY_QUEUEB_ROUTING_KEY, msg);
                break;
        }
}
```

接下来，我们暴露一个web接口来生产消息：

@Slf4j

@RequestMapping\("rabbitmq"\)

@RestController

public

class

RabbitMQMsgController

{

@Autowired

private

DelayMessageSender sender;

@RequestMapping\("sendmsg"\)

public

void

sendMsg

\(String msg, Integer delayType\)

{ log.info\(

"当前时间：{},收到请求，msg:{},delayType:{}"

,

new

Date\(\), msg, delayType\); sender.sendMsg\(msg, Objects.requireNonNull\(DelayTypeEnum.getDelayTypeEnumByValue\(delayType\)\)\); } }


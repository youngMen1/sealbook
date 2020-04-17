# 1.基本介绍

**概念:**

死信队列 听上去像 消息“死”了     其实也有点这个意思，死信队列  是 当消息在一个队列 因为下列原因：

1. 消息被拒绝（basic.reject/ basic.nack）并且不再重新投递 requeue=false
2. 消息超期 \(rabbitmq  Time-To-Live -&gt;messageProperties.setExpiration\(\)\) 
3. 队列超载

**变成了 “死信” 后    被重新投递（publish）到另一个Exchange   该Exchange 就是DLX 然后该Exchange 根据绑定规则 转发到对应的 队列上  监听该队列就可以重新消费说白了就是没有被消费的消息换个地方重新被消费**

**生产者--&gt;消息 --&gt;交换机--&gt;队列  --&gt;变成死信--&gt;DLX交换机--&gt;队列--&gt;消费者**

# 2.怎么使用

## 2.1.RabbitMqConfig配置

```
/**
     * 死信队列 交换机标识符
     */
    private static final String DEAD_LETTER_QUEUE_KEY = "x-dead-letter-exchange";
    /**
     * 死信队列交换机绑定键标识符
     */
    private static final String DEAD_LETTER_ROUTING_KEY = "x-dead-letter-routing-key";

    /**
     * 死信队列跟交换机类型没有关系 不一定为directExchange  不影响该类型交换机的特性.
     *
     * @return the exchange
     */
    @Bean("deadLetterExchange")
    public Exchange deadLetterExchange() {
        return ExchangeBuilder.directExchange("DL_EXCHANGE").durable(true).build();
    }

    /**
     * 声明一个死信队列.
     * x-dead-letter-exchange   对应  死信交换机
     * x-dead-letter-routing-key  对应 死信队列
     *
     * @return the queue
     */
    @Bean("deadLetterQueue")
    public Queue deadLetterQueue() {
        Map<String, Object> args = new HashMap<>(16);
        // x-dead-letter-exchange    声明  死信交换机
        args.put(DEAD_LETTER_QUEUE_KEY, "DL_EXCHANGE");
        // x-dead-letter-routing-key    声明 死信路由键
        args.put(DEAD_LETTER_ROUTING_KEY, "KEY_R");
        return QueueBuilder.durable("DL_QUEUE").withArguments(args).build();
    }

    /**
     * 定义死信队列转发队列.
     *
     * @return the queue
     */
    @Bean("redirectQueue")
    public Queue redirectQueue() {
        return QueueBuilder.durable("REDIRECT_QUEUE").build();
    }

    /**
     * 死信路由通过 DL_KEY 绑定键绑定到死信队列上.
     *
     * @return the binding
     */
    @Bean
    public Binding deadLetterBinding() {
        return new Binding("DL_QUEUE", Binding.DestinationType.QUEUE, "DL_EXCHANGE", "DL_KEY", null);

    }

    /**
     * 死信路由通过 KEY_R 绑定键绑定到死信队列上.
     *
     * @return the binding
     */
    @Bean
    public Binding redirectBinding() {
        return new Binding("REDIRECT_QUEUE", Binding.DestinationType.QUEUE, "DL_EXCHANGE", "KEY_R", null);
    }
```

## 说明：

deadLetterExchange\(\)声明了一个Direct 类型的Exchange （死信队列跟交换机没有关系）

deadLetterQueue\(\) 声明了一个队列   这个队列 跟前面我们声明的队列不一样    注入了 Map&lt;String,Object&gt; 参数    下面的概念非常重要

**x-dead-letter-exchange 来标识一个交换机  x-dead-letter-routing-key  来标识一个绑定键（RoutingKey）  这个绑定键 是分配给 标识的交换机的   如果没有特殊指定 声明队列的原routingkey , 如果有队列通过此绑定键 绑定到交换机    那么死信会被该交换机转发到 该队列上  通过监听 可对消息进行消费  **

可以打个比方  这个是为主力队员 设置了一个替补   如果主力 “死”了   他的活 替补接手  这样更好理解

deadLetterBinding\(\) 对这个带参队列 进行了 和交换机的规则绑定   等下 消费者 先把消息通过交换机投递到该队列中去   然后制造条件发生“死信”

redirectBinding\(\) 我们需要给标识的交换机  以及对其指定的routingkey 来绑定一个所谓的“替补”队列 用来监听

流程具体是  消息投递到  **DL\_QUEUE**  10秒后消息过期 生成死信    然后转发到 **REDIRECT\_QUEUE **通过对其的监听  来消费消息

## 测试

```
/**
     * 案例二、测试死信队列.
     *
     * @param user
     * @return ServerResponse
     */
    @PostMapping("/dead")
    public ServerResponse deadLetter(@RequestBody @Validated User user) {
        rabbitMqService.sendMessage(user);
        return ServerResponse.success();
    }


    public void sendMessage(User user) {
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        /**
         * 声明消息处理器  这个对消息进行处理  可以设置一些参数
         * 对消息进行一些定制化处理   我们这里  来设置消息的编码  以及消息的过期时间
         * 因为在.net 以及其他版本过期时间不一致   这里的时间毫秒值 为字符串
         */
        MessagePostProcessor messagePostProcessor = message -> {
            MessageProperties messageProperties = message.getMessageProperties();
            // 设置编码
            messageProperties.setContentEncoding("utf-8");
            // 设置过期时间10*1000毫秒
            messageProperties.setExpiration("10000");
            return message;
        };
        // 向DL_QUEUE 发送消息  10*1000毫秒后过期 形成死信
        rabbitTemplate.convertAndSend("DL_EXCHANGE", "DL_KEY", JsonUtil.objToStr(user), messagePostProcessor, correlationData);
    }
```

# 微信截图\_20200417113502.png

# 3.总结

# 4.参考

[https://my.oschina.net/10000000000/blog/1626278](https://my.oschina.net/10000000000/blog/1626278)


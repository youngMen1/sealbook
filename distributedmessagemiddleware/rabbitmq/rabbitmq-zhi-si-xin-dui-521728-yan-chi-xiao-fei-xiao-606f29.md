# 1.基本介绍

**概念:**

死信队列 听上去像 消息“死”了     其实也有点这个意思，死信队列  是 当消息在一个队列 因为下列原因：

1. 消息被拒绝（basic.reject/ basic.nack）并且不再重新投递 requeue=false
2. 消息超期 \(rabbitmq  Time-To-Live -&gt;messageProperties.setExpiration\(\)\) 
3. 队列超载

**变成了 “死信” 后    被重新投递（publish）到另一个Exchange   该Exchange 就是DLX 然后该Exchange 根据绑定规则 转发到对应的 队列上  监听该队列就可以重新消费说白了就是没有被消费的消息换个地方重新被消费**

**生产者--&gt;消息 --&gt;交换机--&gt;队列  --&gt;变成死信--&gt;DLX交换机--&gt;队列--&gt;消费者**

# 2.怎么使用



# 3.总结

# 4.参考

[https://my.oschina.net/10000000000/blog/1626278](https://my.oschina.net/10000000000/blog/1626278)


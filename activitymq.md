[https://yq.aliyun.com/articles/652745](https://yq.aliyun.com/articles/652745)

消息队列中间件是分布式系统中重要的组件，主要解决应用耦合，异步消息，流量削锋等问题。实现高性能，高可用，可伸缩和最终一致性架构；是大型分布式系统不可缺少的中间件。目前使用较多的消息队列有ActiveMQ、RabbitMQ、Kafka、RocketMQ、MetaMQ等。springboot提供了对JMS系统的支持；springboot很方便就可以集成这些消息中间件。

对于异步消息在实际的应用之中会有两类：

JMS：代表作就是 ActiveMQ，但是其性能不高，因为其是用 java 程序实现的。

AMQP：直接利用协议实现的消息组件，其大众代表作为RabbitMQ，高性能代表作为Kafka。


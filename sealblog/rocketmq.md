## 描述:

RocketMQ是阿里开源的消息中间件，它是纯Java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。RocketMQ思路起源于Kafka，但并不是Kafka的一个Copy，它对消息的可靠传输及事务性做了优化，目前在阿里集团被广泛应用于交易、充值、流计算、消息推送、日志流式处理、binglog分发等场景。

RocketMQ 3.0和MetaQ 3.0的区别其实这两者是等价的版本，只不过阿里内部使用的称为MetaQ 3.0，外部开源称之为RocketMQ 3.0。

阿里巴巴内部围绕着RocketMQ内核打造了三款产品，分别是MetaQ、Notify和Aliware MQ。这三者分别采用了不同的模型，MetaQ主要使用了拉模型，解决了顺序消息和海量堆积问题；Notify主要使用了推模型，解决了事务消息；而云产品Aliware MQ则是提供了商业化的版本。

## 参考:

https://yq.aliyun.com/articles/71889?spm=5176.10695662.1996646101.searchclickresult.2aa8a3dbOaALG0




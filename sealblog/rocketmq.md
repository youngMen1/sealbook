## 描述:

RocketMQ是阿里开源的消息中间件，它是纯Java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。RocketMQ思路起源于Kafka，但并不是Kafka的一个Copy，它对消息的可靠传输及事务性做了优化，目前在阿里集团被广泛应用于交易、充值、流计算、消息推送、日志流式处理、binglog分发等场景。

RocketMQ 3.0和MetaQ 3.0的区别其实这两者是等价的版本，只不过阿里内部使用的称为MetaQ 3.0，外部开源称之为RocketMQ 3.0。

阿里巴巴内部围绕着RocketMQ内核打造了三款产品，分别是MetaQ、Notify和Aliware MQ。这三者分别采用了不同的模型，MetaQ主要使用了拉模型，解决了顺序消息和海量堆积问题；Notify主要使用了推模型，解决了事务消息；而云产品Aliware MQ则是提供了商业化的版本。

![](/assets/微信截图_20190727110350.png)

**RocketMQ的整体架构设计**

下图为大家清晰地展示了RocketMQ的几个组件，分别是nameserver、broker、producer和consumer。nameserver主要负责对于源数据的管理，包括了对于Topic和路由信息的管理，broker在启动的时候会去向nameserver注册并且定时发送心跳，producer在启动的时候会到nameserver上去拉取Topic所属的broker具体地址，然后向具体的broker发送消息。

![](/assets/微信截图_20190727111204.png)

### 多协议支持 {#h3-u591Au534Fu8BAEu652Fu6301}

* 支持[HTTP 协议](https://help.aliyun.com/document_detail/102996.html)：采用 RESTful 标准，方便易用，快速接入，跨网络能力强，并支持七种语言客户端。
* 支持[TCP 协议](https://help.aliyun.com/document_detail/44711.html)：区别于 HTTP 简单的接入方式，提供更为专业、可靠、稳定的 TCP 协议的 SDK 接入。
* 支持[STOMP 协议](https://help.aliyun.com/document_detail/112558.html)：类似于 HTTP 的纯文本的协议机制，常用于脚本语言（如 Ruby、Python、Perl）和消息队列 RocketMQ Broker 进行轻量级交互。

### 特色功能 {#h3-u7279u8272u529Fu80FD}

* 事务消息：实现类似 X/Open XA 的分布事务功能，以达到事务最终一致性状态。
* 定时（延时）消息：允许消息生产者指定消息进行定时（延时）投递，最长支持 40 天。
* 大消息：支持最大 4 MB 消息。
* 消息轨迹：通过消息轨迹，能清晰定位消息从发布者发出，经由消息队列 RocketMQ 服务端，投递给消息订阅者的完整链路，方便定位排查问题。
* 广播消费：允许同一个 Group ID 所标识的所有 Consumer 都各自消费某条消息一次。
* 顺序消息：允许消息消费者按照消息发送的顺序对消息进行消费。
* 重置消费进度：根据时间重置消费进度，允许用户进行消息回溯或者丢弃堆积消息。
* 死信队列：将无法正常消费的消息储存到特殊的死信队列供后续处理。
* 全球消息路由：用于全球不同地域之间的消息同步复制，保证地域之间的数据一致性。

## 消息收发模型 {#h2-u6D88u606Fu6536u53D1u6A21u578B2}

消息队列 RocketMQ 支持“发布/订阅”模型，消息发布者（生产者）可以将一条消息发送服务端的某个主题（Topic），多个消息接收方（消费者）订阅这个主题以接收该消息，如下图所示：

![](/assets/微信截图_20190727111724.png)

## 参考:

[https://yq.aliyun.com/articles/71889?spm=5176.10695662.1996646101.searchclickresult.2aa8a3dbOaALG0](https://yq.aliyun.com/articles/71889?spm=5176.10695662.1996646101.searchclickresult.2aa8a3dbOaALG0)

[https://help.aliyun.com/document\_detail/29532.html?spm=5176.10695662.1996646101.searchclickresult.2aa8a3dbOaALG0](https://help.aliyun.com/document_detail/29532.html?spm=5176.10695662.1996646101.searchclickresult.2aa8a3dbOaALG0)

[https://www.jianshu.com/p/8c4c2c2ab62e](https://www.jianshu.com/p/8c4c2c2ab62e)

https://www.jianshu.com/p/5260a2739d80


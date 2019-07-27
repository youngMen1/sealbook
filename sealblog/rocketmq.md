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

## 参考:

[https://yq.aliyun.com/articles/71889?spm=5176.10695662.1996646101.searchclickresult.2aa8a3dbOaALG0](https://yq.aliyun.com/articles/71889?spm=5176.10695662.1996646101.searchclickresult.2aa8a3dbOaALG0)

https://help.aliyun.com/document\_detail/29532.html?spm=5176.10695662.1996646101.searchclickresult.2aa8a3dbOaALG0


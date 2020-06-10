# 1.基本介绍

## 1.1.消息队列概述 {#1}

消息队列中间件是分布式系统中重要的组件，主要解决应用解耦，异步消息，流量削锋等问题，实现高性能，高可用，可伸缩和最终一致性架构。目前使用较多的消息队列有ActiveMQ，RabbitMQ，ZeroMQ，Kafka，MetaMQ，RocketMQ

## 1.2.消息队列应用场景 {#2}

以下介绍消息队列在实际应用中常用的使用场景。异步处理，应用解耦，流量削锋和消息通讯四个场景。

### 1.2.1.异步处理 {#3}

场景说明：用户注册后，需要发注册邮件和注册短信。传统的做法有两种 1.串行的方式；2.并行方式

1.串行方式：将注册信息写入数据库成功后，发送注册邮件，再发送注册短信。以上三个任务全部完成后，返回给客户端。

### 1.2.2.应用解耦 {#4}

### 1.2.3.流量削锋 {#5}

### 1.2.4.日志处理 {#6}

### 1.2.5.消息通讯 {#7}

## 1.3.消息中间件示例 {#8}

# 2.参考:

Java消息队列总结只需一篇解决ActiveMQ、RabbitMQ、ZeroMQ、Kafka

[https://yq.aliyun.com/articles/661488?spm=a2c4e.11153940.0.0.59e644d2Q1YNVY](https://yq.aliyun.com/articles/661488?spm=a2c4e.11153940.0.0.59e644d2Q1YNVY)

消息中间件ActiveMQ、RabbitMQ、RocketMQ、ZeroMQ、Kafka如何选型？

[https://yq.aliyun.com/articles/619292?spm=a2c4e.11153940.0.0.219638b0RfjjN2](https://yq.aliyun.com/articles/619292?spm=a2c4e.11153940.0.0.219638b0RfjjN2)

RabbitMQ和Kafka成熟度评测

[https://mp.weixin.qq.com/s/2Esqohw8L30Yvw4Dmr53nA?](https://mp.weixin.qq.com/s/2Esqohw8L30Yvw4Dmr53nA?)


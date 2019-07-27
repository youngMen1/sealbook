## metaMQ是什么?

基于Kafka的设计使用Java进行了完全重写并推出了MetaQ 1.0版本

现在metaq其实有两个大分支了，一个是庄晓丹维护的已开源的，另外一个是淘宝内部的，本质结构原理没太大区别，只不过开源的已经去掉了对淘系相关的依赖。然后淘系的metaq已经到3.\*版本了

metaMq是一个分布式消息中间件，消息中间件是典型的生产者-消费者模型，核心作用是解耦，生产者和消费者彼此没有直接依赖，同步化解成了异步。metaq并没有遵循jms规范，jms规范体现在系统层面和api层面。

## 消费模型

例如jms定义了两种消息传递方式：

1.基于队列的点对点消费模型

2.基于发布/订阅的消费模型

**metaMq只有发布订阅的消费方式。**

## 消息类型

JMS定义的消息类型有TextMessage、MapMessage、BytesMessage、StreamMessage、ObjectMessage。Metaq只有一种类型：Message。

## 消息持久性

JMS定义两种持久性类型：

PERSISTENT 指示JMS provider持久保存消息，以保证消息不会因为JMS provider的失败而丢失。  
NON\_PERSISTENT 不要求JMS provider持久保存消息。

metaMq的消息都是持久性的

## API

JMS定义了消息中间件的生产端api和消费端api，这些api都是约定的接口，都都被metaMq无视了。

## 概念

#### 消息生产者

负责产生消息并发送消息到meta服务器

#### 消息消费者

负责消息的消费，meta采用pull模型，由消费者主动从meta服务器拉取数据并解析成消息并消费

#### Topic

消息的主题，由用户定义并在服务端配置。producer发送消息到某个topic下，consumer从某个topic下消费消息

#### Partition\(分区\)

同一个topic下面还分为多个分区，如meta-test这个topic我们可以分为10个分区，分别有两台服务器提供，那么可能每台服务器提供5个分 区，假设服务器分别为0和1，则所有分区为0-0、0-1、0-2、0-3、0-4、1-0、1-1、1-2、1-3、1-4

#### Message

消息，负载用户数据并在生产者、服务端和消费者之间传输

#### Broker

就是meta的服务端或者说服务器，在消息中间件中也通常称为broker。

#### 消费者分组\(Group\)

消费者可以是多个消费者共同消费一个topic下的消息，每个消费者消费部分消息。这些消费者就组成一个分组，拥有同一个分组名称,通常也称为消费者集群

#### Offset

消息在broker上的每个分区都是组织成一个文件列表，消费者拉取数据需要知道数据在文件中的偏移量，这个偏移量就是所谓offset。Offset是绝对偏移量，服务器会将offset转化为具体文件的相对偏移量

## 总体架构图

![](/assets/微信截图_20190727095239.png)

## **消息存储**

消息中间件中消息堆积是很常见，这要求broker具有消息存储的能力，消息存储结构决定了消息的读写性能，对整体性能有很大影响，metaq是分布式的，多个borker可以为一个topic提供服务，一个topic下的消息分散存储在多个broker，它们是多对多关系。

![](/assets/微信截图_20190727095341.png)

## 消息定义

id

消息的唯一id，系统自动产生，用户无法设置，在发送成功后由服务器返回，发送失败则为0。

topic

消息的主题，订阅者订阅该主题即可接收发送到该主题下的消息，必须

data

消息的有效载荷，也就是消息内容，meta永远不会修改消息内容，你发送出去是什么样子，接收到就是什么样子。

attribute

消息属性，一个字符串，可选。发送者可设置消息属性来让消费者过滤。

## 逻辑结构图

![](/assets/微信截图_20190727095626.png)

## MetaQ适合的场景？

日志传输，高吞吐量的日志传输，这本来也是kafka的强项。

消息广播功能，如广播缓存配置失效。

数据的顺序同步功能，如MySQL binlog复制。

分布式环境下（broker、producer、consumer都为集群）的消息路由，对顺序和可靠性有极高要求的场景。

作为一般MQ来使用的其他功能。

---

## 参考:

[https://blog.csdn.net/frank1998819/article/details/84767357](https://blog.csdn.net/frank1998819/article/details/84767357)

[https://blog.csdn.net/blogdevteam/article/details/8449916](https://blog.csdn.net/blogdevteam/article/details/8449916)


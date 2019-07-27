## MetaMQ是什么?

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



## 参考:

[https://blog.csdn.net/frank1998819/article/details/84767357](https://blog.csdn.net/frank1998819/article/details/84767357)


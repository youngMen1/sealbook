## 描述:

Kafka是linkedin开源的MQ系统，主要特点是基于Pull的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输，0.8开始支持复制，不支持事务，适合产生大量数据的互联网服务的数据收集业务。

Apache Kafka 是一个分布式流处理平台，用于构建实时的数据管道和流式的应用.它可以让你发布和订阅流式的记录，可以储存流式的记录，并且有较好的容错性，可以在流式记录产生时就进行处理。

Apache Kafka是分布式发布-订阅消息系统，在 kafka官网上对 Kafka 的定义：一个分布式发布-订阅消息传递系统。

#### Kafka 特性

1. 高吞吐量、低延迟：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多个partition, consumer group 对partition进行consume操作；
2. 可扩展性：kafka集群支持热扩展；
3. 持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失；
4. 容错性：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）；
5. 高并发：支持数千个客户端同时读写；
6. 支持实时在线处理和离线处理：可以使用Storm这种实时流处理系统对消息进行实时进行处理，同时还可以使用Hadoop这种批处理系统进行离线处理；

#### Kafka 使用场景

1. 日志收集：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如Hadoop、Hbase、Solr等；
2. 消息系统：解耦和生产者和消费者、缓存消息等；
3. 用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到Hadoop、数据仓库中做离线分析和挖掘；
4. 运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告；
5. 流式处理：比如spark streaming和storm；
6. 事件源；

## kafka可视化客户端工具

下载地址:

```
http://www.kafkatool.com/download.html
```

安装:

![](/assets/微信截图_20190725150010.png)



简单使用

```
https://www.cnblogs.com/frankdeng/p/9403883.html
```





## 注意:

kafka服务器的端口要开放，可以访问该端口

host文件中需要添加服务器的ip和hostname

C:\Windows\System32\drivers\etc

192.168.23.128 centos101

192.168.23.132 slave01

106.12.40.240 instance-6orhoahv

## 参考:

[http://kafka.apache.org/](http://kafka.apache.org/)

[https://www.jianshu.com/p/04eff11430e4](https://www.jianshu.com/p/04eff11430e4)

kfkaTool：[http://www.kafkatool.com/download.html](http://www.kafkatool.com/download.html)


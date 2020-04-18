# 1.基本介绍

## 1.1.Apollo简介

Apollo（阿波罗）是携程框架部门研发的开源配置管理中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性。

**Apollo支持4个维度管理Key-Value格式的配置：**

* application（应用级别的，针对于当前应用）
* environment（环境级别的，在不同环境下可以有不同的配置。比如dev、prod环境）
* cluster（集群。可以根据集群分为不同的配置）
* namespace（命名空间。不同的配置文件，比如db的配置，rpc的配置等等） 

**Apollo基于开源模式开发，开源地址：**[https://github.com/ctripcorp/apollo](https://github.com/ctripcorp/apollo)

## 1.2.Apollo的特性

可以统一去管理不同环境、不同集群的配置。

（1）同一份代码部署在不同的集群，可以有不同的配置。

（2）可以通过配置namespace去让不同的应用共享同一份配置（public公共配置），也可以通过关联公共的namespace，并且根据自己的需求对公共配置进行覆盖。（相当于子类可以继承父类，对于父类中的成员，不满足需要的可以自己进行覆盖。）

* 支持热发布。在管理界面修改完配置后，客户端可以实时的接收到。（据它的描述，时间为1s）

* 每一次改动配置进行发布，都会有对应的版本的概念，方便后续进行回滚。

* 灰度发布。（是指改动配置点击发布，只针对部分实例生效。待观察之后，没有问题，再推送到所有实例。）

* 权限管理（可以针对不同的操作，分配以不同的权限。并且在配置的改动发布上，分为编辑和发布两个独立的操作。）

* 支持Java和.Net客户端。

* 提供开放平台API。

* 部署简单方便（目前唯一的外部依赖是MySQL，因此安装好JDK和MySQL就可以运行。）

# 参考

携程Apollo统一配置中心的搭建和使用：

[https://blog.csdn.net/luhong327/article/details/81453001](https://blog.csdn.net/luhong327/article/details/81453001)

携程 Apollo分布式部署：

[https://www.cnblogs.com/panwenbin-logs/p/10956826.html](https://www.cnblogs.com/panwenbin-logs/p/10956826.html)


# 1.基本介绍

## 1.1.Apollo简介

Apollo（阿波罗）是携程框架部门研发的开源配置管理中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性。

**Apollo支持4个维度管理Key-Value格式的配置：**

* application（应用级别的，针对于当前应用）
* environment（环境级别的，在不同环境下可以有不同的配置。比如dev、prod环境）
* cluster（集群。可以根据集群分为不同的配置）
* namespace（命名空间。不同的配置文件，比如db的配置，rpc的配置等等） 

**Apollo基于开源模式开发，开源地址：**[https://github.com/ctripcorp/apollo](https://github.com/ctripcorp/apollo)

## 

# 参考

携程Apollo统一配置中心的搭建和使用：

[https://blog.csdn.net/luhong327/article/details/81453001](https://blog.csdn.net/luhong327/article/details/81453001)

携程 Apollo分布式部署：

[https://www.cnblogs.com/panwenbin-logs/p/10956826.html](https://www.cnblogs.com/panwenbin-logs/p/10956826.html)


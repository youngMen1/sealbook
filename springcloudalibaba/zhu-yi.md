# 1**、阿里巴巴开源组件**

Spring Cloud Alibaba项目由两部分组成：阿里巴巴开源组件和阿里云产品组件，旨在为Java开发人员在使用阿里巴巴产品的同时，通过利用 Spring 框架的设计模式和抽象能力，注入Spring Boot和Spring Cloud的优势。

注意： 版本 0.2.0.RELEASE 对应的是 Spring Boot 2.x 版本，版本 0.1.0.RELEASE 对应的是 Spring Boot 1.x 版本.

## 1.1、**服务发现**

实现了 Spring Cloud common 中定义的 registry 相关规范接口，引入依赖并添加一些简单的配置即可将你的服务注册到Nacos Server中，并且支持与Ribbon的集成。

## 1.2、**配置管理**

实现了 

`PropertySoureLocator`

 接口，引入依赖并添加一些简单的配置即可从 Nacos Server 中获取应用配置并设置在 Spring 的 Environment 中，而且无需依赖其他组件即可支持配置的实时推送和推送状态查询。




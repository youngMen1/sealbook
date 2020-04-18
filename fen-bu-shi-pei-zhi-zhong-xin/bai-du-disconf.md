## 1.基本介绍

Disconf是百度的一个分布式配置中心,目前已经开源。而且它是基于java实现的,有简单的配置页面，而且官方还提供了一个相对完善的[文档](https://disconf.readthedocs.io/zh_CN/latest/).开发者只需按照它上面的步骤来即可安装

## 1.1.DisConf简介

Distributed Configuration Management Platform\(分布式配置管理平台\)

百度disconf是一套完整的基于zookeeper的分布式配置统一解决方案。

一个分布式环境中，同类型的服务往往会部署很多实例。这些实例使用了一些配置，为了更好地维护这些配置就产生了配置管理服务。通过这个服务可以轻松地管理成千上百个服务实例的配置问题。专注于各种「分布式系统配置管理」的「通用组件」和「通用平台」, 提供统一的「配置管理服务」
![img](/static/image/20180607092944288.png)
**使用公司：** 百度、滴滴出行、银联、网易、拉勾网、苏宁易购、顺丰科技 等知名互联网公司正在使用!
## 1.2.主要目标
* 部署极其简单：同一个上线包，无须改动配置，即可在 多个环境中(RD/QA/PRODUCTION) 上线
* 部署动态化：更改配置，无需重新打包或重启，即可 实时生效
* 统一管理：提供web平台，统一管理 多个环境(RD/QA/PRODUCTION)、多个产品 的所有配置
* 核心目标：一个jar包，到处运行

## 1.3.功能特点

支持配置（配置项+配置文件）的分布式化管理
配置发布统一化

**配置发布、更新统一化:**

同一个上线包 无须改动配置 即可在 多个环境中(RD/QA/PRODUCTION) 上线
配置存储在云端系统，用户统一管理 多个环境(RD/QA/PRODUCTION)、多个平台 的所有配置
**配置更新自动化：**
用户在平台更新配置，使用该配置的系统会自动发现该情况，并应用新配置。特殊地，如果用户为此配置定义了回调函数类，则此函数类会被自动调用。
**极简的使用方式（注解式编程 或 XML无代码侵入模式）：**
我们追求的是极简的、用户编程体验良好的编程方式。目前支持两种开发模式：
基于XML配置或者基于注解，即可完成复杂的配置分布式化。
注：配置项是指某个类里的某个Field字段。
**Disconf的功能特点描述图：**
![img](/static/image/20180607093327344.jpg)

# 4.总结

### 4.1.DisConf-分布式配置管理平台-简介 {#1disconf-分布式配置管理平台-简介}

**github地址：**

[https://github.com/knightliao/disconf](https://github.com/knightliao/disconf)  
**文档地址：**

[http://disconf.readthedocs.io/zh\_CN/latest/](http://disconf.readthedocs.io/zh_CN/latest/)

# 参考

百度DisConf-分布式配置管理平台-安装：  
[https://blog.csdn.net/fd2025/article/details/80607315](https://blog.csdn.net/fd2025/article/details/80607315)

\[分布式配置管理--百度disconf搭建过程和详细使用：  
[https://www.cnblogs.com/garfieldcgf/p/6439221.html](https://www.cnblogs.com/garfieldcgf/p/6439221.html)

Disconf —— 来自百度的分布式配置管理平台：

[https://blog.csdn.net/why\_768/article/details/61419828](https://blog.csdn.net/why_768/article/details/61419828)
百度DisConf-分布式配置管理平台-简介：
https://blog.csdn.net/fd2025/article/details/80604802



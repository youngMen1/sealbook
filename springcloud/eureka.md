# 1.基本介绍

[Eureka](https://github.com/Netflix/Eureka) 是 [Netflix](https://github.com/Netflix) 开发的，一个基于 REST 服务的，服务注册与发现的组件

它主要包括两个组件：Eureka Server 和 Eureka Client

* Eureka Client：一个Java客户端，用于简化与 Eureka Server 的交互（通常就是微服务中的客户端和服务端）
* Eureka Server：提供服务注册和发现的能力（通常就是微服务中的注册中心）
  ![img](/static/image/398358-20190722105850485-951984065.png)
  各个微服务启动时，会通过 Eureka Client 向 Eureka Server 注册自己，Eureka Server 会存储该服务的信息
  也就是说，每个微服务的客户端和服务端，都会注册到 Eureka Server，这就衍生出了微服务相互识别的话题
  同步：每个 Eureka Server 同时也是 Eureka Client（逻辑上的）
  　　　多个 Eureka Server 之间通过复制的方式完成服务注册表的同步，形成 Eureka 的高可用
  识别：Eureka Client 会缓存 Eureka Server 中的信息
  　　　即使所有 Eureka Server 节点都宕掉，服务消费者仍可使用缓存中的信息找到服务提供者（笔者已亲测）
  续约：微服务会周期性（默认30s）地向 Eureka Server 发送心跳以Renew（续约）信息（类似于heartbeat）
  续期：Eureka Server 会定期（默认60s）执行一次失效服务检测功能
  　　　它会检查超过一定时间（默认90s）没有Renew的微服务，发现则会注销该微服务节点
  Spring Cloud 已经把 Eureka 集成在其子项目 Spring Cloud Netflix 里面

# 2.怎么使用

## 服务端配置

**@EnableEurekaServer：服务注册中心\(服务端\)**，这个注解需要在springboot工程的启动application类上加

```
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

spring:
  application:
    name: eurka-server
```

## 客户端配置

**@EnableEurekaClient：**开启客户端，这个注解需要在springboot工程的启动application类上加

```
server:
  port: 8762

spring:
  application:
    name: service-hi

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

## 结果

![img](/static/image/2279594-d830f93f1e56f6a2.png)

## eureka健康检查

health 健康检查，修改访问路径  
  2.0 之前默认是/  
  2.0 默认是 /actuator 可以通过这个属性值修改

```
#health 健康检查
management:
  endpoints:
   web:
     #开放所有页面节点  默认只开启了health、info两个节点
     exposure:
       include: "*"
  #显示健康具体信息  默认不会显示详细信息
  endpoint:
    health:
      show-details: always
```

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

# 3.总结

spring cloud中discovery service有许多种实现（eureka、consul、zookeeper等等），@EnableDiscoveryClient基于spring-cloud-commons, @EnableEurekaClient基于spring-cloud-netflix。

**其实用更简单的话来说，就是**

**如果选用的注册中心是eureka，那么就推荐@EnableEurekaClient，**

**如果是其他的注册中心，那么推荐使用@EnableDiscoveryClient。**

## eureka踩坑日志

1.从Eureka注册中心删除服务\(PostMan delete请求\)

[http://:eureka\_center/eureka/apps/:application/:instance](http://:eureka_center/eureka/apps/:application/:instance)

```
## 例如
http://xxx.xx.xx.xxx:xxxxx/eureka/apps/HOTEL-REST/UTECHNO9.utech.com:hotel-rest:xxxx
```

2.服务提供者使用主机名注册到eureka改为使用ip注册到eureka

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10010/eureka/
  instance:
    preferIpAddress : true
    instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}
```

其中获取ip，SpringCloud2.0版本对应的key值为${spring.cloud.clent.ip-address},网上流传大多为${spring.cloud.clent.ipAddress}

# 4.参考

官网项目地址：[https://github.com/spring-cloud/spring-cloud-netflix/issues/203](https://github.com/spring-cloud/spring-cloud-netflix/issues/203)

[Eureka - Eureka配置列表](https://www.cnblogs.com/tiancai/p/9593648.html)：[https://www.cnblogs.com/tiancai/p/9593648.html](https://www.cnblogs.com/tiancai/p/9593648.html)

[Eureka的自我保护模式、IP选择、健康检查](https://www.cnblogs.com/jinjiyese153/p/8617951.html)：[https://www.cnblogs.com/jinjiyese153/p/8617951.html](https://www.cnblogs.com/jinjiyese153/p/8617951.html)

@EnableEurekaServer注解源码: [https://blog.csdn.net/TP89757/article/details/100877037](https://blog.csdn.net/TP89757/article/details/100877037)

[https://www.jianshu.com/p/1aadc4c85f51](#)


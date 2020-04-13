# 1.基本介绍

[Eureka](https://github.com/Netflix/Eureka) 是 [Netflix](https://github.com/Netflix) 开发的，一个基于 REST 服务的，服务注册与发现的组件

它主要包括两个组件：Eureka Server 和 Eureka Client

* Eureka Client：一个Java客户端，用于简化与 Eureka Server 的交互（通常就是微服务中的客户端和服务端）
* Eureka Server：提供服务注册和发现的能力（通常就是微服务中的注册中心）
![img](/static/image/398358-20190722105850485-951984065.png)


# 2.怎么使用

# 3.总结

# 4.参考

# 

# 

# SpringBoot 2.0以上

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

## 参考:

[https://www.jianshu.com/p/1aadc4c85f51](https://www.jianshu.com/p/1aadc4c85f51)


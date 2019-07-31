## 

SpringBoot 2.0之后配置

## eureka健康检查

health 健康检查，修改访问路径  
  2.0之前默认是/  
  2.0默认是 /actuator 可以通过这个属性值修改

```
management:
  endpoints:
   web:
     base-path: /
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


## eureka踩坑日志

1.从Eureka注册中心删除服务\(PostMan\)

[http://:eureka\_center/eureka/apps/:application/:instance](http://:eureka_center/eureka/apps/:application/:instance)

例如:[http://xxx.xx.xx.xxx:xxxxx/eureka/apps/HOTEL-REST/UTECHNO9.utech.com:hotel-rest:](http://172.18.21.243:22002/eureka/apps/HOTEL-REST/UTECHNO9.utech.com:hotel-rest:9700)xxxx

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


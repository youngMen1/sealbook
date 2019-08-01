## 一、Feign简介

Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。  
简而言之:  
Feign 采用的是基于接口的注解  
Feign 整合了ribbon，具有负载均衡的能力  
整合了Hystrix，具有熔断的能力:

```
feign.hystrix.enabled=true
```

## 二、Feign的工作原理

feign是一个伪客户端，即它不做任何的请求处理。Feign通过处理注解生成request，从而实现简化HTTP API开发的目的，即开发人员可以使用注解的方式定制request api模板，在发送http request请求之前，feign通过处理注解的方式替换掉request模板中的参数，这种实现方式显得更为直接、可理解。通过包扫描注入FeignClient的bean，该源码在FeignClientsRegistrar类：首先在启动配置上检查是否有@EnableFeignClients注解，如果有该注解，则开启包扫描，扫描被@FeignClient注解接口




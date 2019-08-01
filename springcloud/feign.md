## 一、Feign简介

Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。  
简而言之:  
Feign 采用的是基于接口的注解  
Feign 整合了ribbon，具有负载均衡的能力  
整合了Hystrix，具有熔断的能力:

```
feign.hystrix.enabled=true
```

Spring Cloud feign使用中在使用服务发现的时候提到了两种注解，一种为@EnableDiscoveryClient,一种为@EnableEurekaClient,用法上基本一致。spring cloud中discovery service有许多种实现（eureka、consul、zookeeper等等），@EnableDiscoveryClient基于spring-cloud-commons, @EnableEurekaClient基于spring-cloud-netflix。其实用更简单的话来说，就是如果选用的注册中心是eureka，那么就推荐@EnableEurekaClient，如果是其他的注册中心，那么推荐使用@EnableDiscoveryClient。

## 二、Feign的工作原理

feign是一个伪客户端，即它不做任何的请求处理。Feign通过处理注解生成request，从而实现简化HTTP API开发的目的，即开发人员可以使用注解的方式定制request api模板，在发送http request请求之前，feign通过处理注解的方式替换掉request模板中的参数，这种实现方式显得更为直接、可理解。

通过包扫描注入FeignClient的bean，该源码在FeignClientsRegistrar类：  
首先在启动配置上检查是否有@EnableFeignClients注解，如果有该注解，则开启包扫描，扫描被@FeignClient注解接口

```
private void registerDefaultConfiguration(AnnotationMetadata metadata,
            BeanDefinitionRegistry registry) {
        Map<String, Object> defaultAttrs = metadata
                .getAnnotationAttributes(EnableFeignClients.class.getName(), true);

        if (defaultAttrs != null && defaultAttrs.containsKey("defaultConfiguration")) {
            String name;
            if (metadata.hasEnclosingClass()) {
                name = "default." + metadata.getEnclosingClassName();
            }
            else {
                name = "default." + metadata.getClassName();
            }
            registerClientConfiguration(registry, name,
                    defaultAttrs.get("defaultConfiguration"));
        }
    }
```

如果想要feign使用Okhttp，则只需要在pom文件上加上feign-okhttp的依赖：

```
<dependency>
    <groupId>com.netflix.feign</groupId>
    <artifactId>feign-okhttp</artifactId>
    <version>RELEASE</version>
</dependency>
```

## 总结

总到来说，Feign的源码实现的过程如下：

首先通过@EnableFeignCleints注解开启FeignCleint

根据Feign的规则实现接口，并加@FeignCleint注解

程序启动后，会进行包扫描，扫描所有的@ FeignCleint的注解的类，并将这些信息注入到ioc容器中。

当接口的方法被调用，通过jdk的代理，来生成具体的RequesTemplate

RequesTemplate在生成Request

Request交给Client去处理，其中Client可以是HttpUrlConnection、HttpClient也可以是Okhttp

最后Client被封装到LoadBalanceClient类，这个类结合类Ribbon做到了负载均衡。

---

## 参考

[https://blog.csdn.net/forezp/article/details/73480304](https://blog.csdn.net/forezp/article/details/73480304)


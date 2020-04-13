# 1.基本介绍

## 1.1.Spring Cloud gateway工作流程

网关作为一个系统的流量的入口，有着举足轻重的作用，通常的作用如下：

* 协议转换，路由转发
* 流量聚合，对流量进行监控，日志输出
* 作为整个系统的前端工程，对流量进行控制，有限流的作用
* 作为系统的前端边界，外部流量只能通过网关才能访问系统
* 可以在网关层做权限的判断
* 可以在网关层做缓存
  Spring Cloud Gateway作为Spring Cloud框架的第二代网关，在功能上要比Zuul更加的强大，性能也更好。随着Spring Cloud的版本迭代，Spring Cloud官方有打算弃用Zuul的意思。在笔者调用了Spring Cloud Gateway的使用和功能上，Spring Cloud Gateway替换掉Zuul的成本上是非常低的，几乎可以无缝切换。Spring Cloud Gateway几乎包含了zuul的所有功能。
  ![img](/static/image/spring_cloud_gateway_diagram.png)

如上图所示，客户端向Spring Cloud Gateway发出请求。 如果Gateway Handler Mapping确定请求与路由匹配（这个时候就用到predicate），则将其发送到Gateway web handler处理。 Gateway web handler处理请求时会经过一系列的过滤器链。 过滤器链被虚线划分的原因是过滤器链可以在发送代理请求之前或之后执行过滤逻辑。 先执行所有“pre”过滤器逻辑，然后进行代理请求。 在发出代理请求之后，收到代理服务的响应之后执行“post”过滤器逻辑。这跟zuul的处理过程很类似。在执行所有“pre”过滤器逻辑时，往往进行了鉴权、限流、日志输出等功能，以及请求头的更改、协议的转换；转发之后收到响应之后，会执行所有“post”过滤器的逻辑，在这里可以响应数据进行了修改，比如响应头、协议的转换等。

在上面的处理过程中，有一个重要的点就是讲请求和路由进行匹配，这时候就需要用到predicate，它是决定了一个请求走哪一个路由。

## 1.2.predicate简介

```
Predicate来自于java8的接口。Predicate 接受一个输入参数，返回一个布尔值结果。
该接口包含多种默认方法来将Predicate组合成其他复杂的逻辑（比如：与，或，非）。
可以用于接口请求参数校验、判断新老数据是否有变化需要进行更新操作。add--与、or--或、negate--非。
```

Spring Cloud Gateway内置了许多Predict,这些Predict的源码在org.springframework.cloud.gateway.handler.predicate包中，如果读者有兴趣可以阅读一下。现在列举各种Predicate如下图：

![img](/static/image/12191355-7c74ff861a209cd9.png)

在上图中，有很多类型的Predicate,比如说

时间类型的Predicated（AfterRoutePredicateFactory BeforeRoutePredicateFactory BetweenRoutePredicateFactory），当只有满足特定时间要求的请求会进入到此predicate中，并交由router处理；

cookie类型的CookieRoutePredicateFactory，指定的cookie满足正则匹配，才会进入此router;

以及host、method、path、querparam、remoteaddr类型的predicate，

每一种predicate都会对当前的客户端请求进行判断，是否满足当前的要求，

如果满足则交给当前请求处理。如果有很多个Predicate，并且一个请求满足多个Predicate，则按照配置的顺序第一个生效。

# 2.怎么使用

## 2.1.predicate配置

### After Route Predicate Factory {#after-route-predicate-factory}

AfterRoutePredicateFactory，可配置一个时间，当请求的时间在配置时间之后，才交给 router去处理。否则则报错，不通过路由。

在工程的application.yml配置如下：

```
server:
  port: 8081
spring:
  profiles:
    active: after_route

---
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: http://httpbin.org:80/get
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
  profiles: after_route
```

在上面的配置文件中，配置了服务的端口为8081，配置spring.profiles.active:after\_route指定了程序的spring的启动文件为after\_route文件。在application.yml再建一个配置文件，语法是三个横线，在此配置文件中通过spring.profiles来配置文件名，和spring.profiles.active一致，然后配置spring cloud gateway 相关的配置，id标签配置的是router的id，每个router都需要一个唯一的id，uri配置的是将请求路由到哪里，本案例全部路由到[http://httpbin.org:80/get。](http://httpbin.org:80/get。)

predicates： After=2017-01-20T17:42:47.789-07:00\[America/Denver\] 会被解析成PredicateDefinition对象 （name =After ，args= 2017-01-20T17:42:47.789-07:00\[America/Denver\]）。**在这里需要注意的是predicates的After这个配置，遵循的契约大于配置的思想，它实际被AfterRoutePredicateFactory这个类所处理**，这个After就是指定了它的Gateway web handler类为AfterRoutePredicateFactory，同理，其他类型的predicate也遵循这个规则。

当请求的时间在这个配置的时间之后，请求会被路由到[http://httpbin.org:80/get。](http://httpbin.org:80/get。)

启动工程，在浏览器上访问[http://localhost:8081/，会显示http://httpbin.org:80/get返回的结果，此时gateway路由到了配置的uri。如果我们将配置的时间设置到当前时之后，浏览器会显示404，此时证明没有路由到配置的uri](http://localhost:8081/，会显示http://httpbin.org:80/get返回的结果，此时gateway路由到了配置的uri。如果我们将配置的时间设置到当前时之后，浏览器会显示404，此时证明没有路由到配置的uri).

跟时间相关的predicates还有Before Route Predicate Factory、Between Route Predicate Factory，读者可以自行查阅官方文档，再次不再演示。

### Header Route Predicate Factory {#header-route-predicate-factory}

Header Route Predicate Factory需要2个参数，一个是header名，另外一个header值，该值可以是一个正则表达式。当此断言匹配了请求的header名和值时，断言通过，进入到router的规则中去。

# 3.总结

# 4.参考资料

[https://www.fangzhipeng.com/springcloud/2018/12/05/sc-f-gateway2.html](https://www.fangzhipeng.com/springcloud/2018/12/05/sc-f-gateway2.html)

**官方文档地址：**[http://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.0.0.RELEASE/single/spring-cloud-gateway.html](http://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.0.0.RELEASE/single/spring-cloud-gateway.html)


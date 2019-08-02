在微服务架构中，需要几个基础的服务治理组件，包括服务注册与发现、服务消费、负载均衡、断路器、智能路由、配置管理等，由这几个基础组件相互协作，共同组建了一个简单的微服务系统。一个简答的微服务系统如下图：

![](/assets/微信截图_20190802084710.png)

在Spring Cloud微服务系统中，一种常见的负载均衡方式是，客户端的请求首先经过负载均衡（zuul、Ngnix），再到达服务网关（zuul集群），然后再到具体的服务，服务统一注册到高可用的服务注册中心集群，服务的所有的配置文件由配置服务管理配置服务的配置文件放在git仓库，方便开发人员随时改配置。

### 一、Zuul简介

Zuul的主要功能是路由转发和过滤器。路由功能是微服务的一部分，比如／api/user转发到到user服务，/api/shop转发到到shop服务。zuul默认和Ribbon结合实现了负载均衡的功能。

zuul有以下功能：

Authentication   认证

Insights  观察

Stress Testing   压力测试

Canary Testing

Dynamic Routing  动态路由

Service Migration 服务迁移

Load Shedding  负载削减

Security 安全

Static Response handling 静态响应处理

Active/Active traffic management 主动/主动流量管理

![](/assets/微信截图_20190802085208.png)

Zuul有四种标准过滤器类型：

* pre - 路由之前            **预**过滤器在请求路由之前执行，

* routing- 路由之时      **路由**过滤器可以处理请求的实际路由，

* post-  路由之后         **邮件**过滤器在请求被路由后执行，并且

* error-  发生错误调用        如果在处理请求过程中发生错误，则会执行**错误**过滤器

过滤器类实现四种方法：

* `filterType()`返回一个`String`代表过滤器类型的东西---在这种情况下`pre`，或者它可以`route`用于路由过滤器。

* `filterOrder()`给出了相对于其他过滤器执行此过滤器的顺序。

* `shouldFilter()`包含确定何时执行此过滤器的逻辑（将_始终_执行此特定过滤器）。

* `run()`包含过滤器的功能。

Zuul过滤器将请求和状态信息存储在（并通过）共享`RequestContext`。我们正在使用它来获取`HttpServletRequest`，然后我们记录请求的HTTP方法和URL，然后再发送它。

## 踩坑日记

# 当stripPrefix=true的时候 （[http://127.0.0.1:8181/api/user/list](http://127.0.0.1:8181/api/user/list) -&gt;[http://192.168.1.100:8080/user/list](http://192.168.1.100:8080/user/list)）

# 当stripPrefix=false的时候（[http://127.0.0.1:8181/api/user/list](http://127.0.0.1:8181/api/user/list) -&gt;[http://192.168.1.100:8080/api/user/list](http://192.168.1.100:8080/api/user/list)）

---

## 参考:

[https://blog.csdn.net/forezp/article/details/81041012](https://blog.csdn.net/forezp/article/details/81041012)


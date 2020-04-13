# 1.基本介绍

## 1.1.filter的作用和生命周期

由filter工作流程点，可以知道filter有着非常重要的作用，

在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，

在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等。首先需要弄清一点为什么需要网关这一层，这就不得不说下filter的作用了。

### 1.2.作用

当我们有很多个服务时，比如下图中的user-service、goods-service、sales-service等服务，客户端请求各个服务的Api时，每个服务都需要做相同的事情，比如鉴权、限流、日志输出等。  
![img](/static/image/2279594-1ad11b98829e33e5.png)

对于这样重复的工作，有没有办法做的更好，答案是肯定的。在微服务的上一层加一个全局的权限控制、限流、日志输出的Api Gatewat服务，然后再将请求转发到具体的业务服务层。这个Api Gateway服务就是起到一个服务边界的作用，外接的请求访问系统，必须先通过网关层。  
![img](/static/image/2279594-8aef2be1eccab3d8.png)
### 1.3.生命周期
Spring Cloud Gateway同zuul类似（前置过滤器和后置过滤器），有“pre”和“post”两种方式的filter。客户端的请求先经过“pre”类型的filter，然后将请求转发到具体的业务服务，比如上图中的user-service，收到业务服务的响应之后，再经过“post”类型的filter处理，最后返回响应到客户端。
![img](/static/image/2279594-16242cf54a5b82e8.png)
与zuul不同的是，filter除了分为“pre”和“post”两种方式的filter外，在Spring Cloud Gateway中，filter从作用范围可分为另外两种，一种是针对于单个路由的gateway filter，它在配置文件中的写法同predict类似；另外一种是针对于所有路由的global gateway filer。现在从作用范围划分的维度来讲解这两种filter。

# 2.怎么使用
## gateway filter
过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。过滤器可以限定作用在某些特定请求路径上。 Spring Cloud Gateway包含许多内置的GatewayFilter工厂。

GatewayFilter工厂同上一篇介绍的Predicate工厂类似，都是在配置文件application.yml中配置，遵循了约定大于配置的思想，只需要在配置文件配置GatewayFilter Factory的名称，而不需要写全部的类名，比如AddRequestHeaderGatewayFilterFactory只需要在配置文件中写AddRequestHeader，而不是全部类名。在配置文件中配置的GatewayFilter Factory最终都会相应的过滤器工厂类处理。
Spring Cloud Gateway 内置的过滤器工厂一览表如下：

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
![img](/static/image/2279594-21f95f970275e70f.png)  
现在挑几个常见的过滤器工厂来讲解，每一个过滤器工厂在官方文档都给出了详细的使用案例，如果不清楚的还可以在org.springframework.cloud.gateway.filter.factory看每一个过滤器工厂的源码。

### AddRequestHeader GatewayFilter Factory {#addrequestheader-gatewayfilter-factory}

入相关的依赖,包括spring boot 版本2.0.5，spring Cloud版本Finchley，gateway依赖如下：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

在工程的配置文件中，加入以下的配置：

```
server:
  port: 8081
spring:
  profiles:
    active: add_request_header_route

---
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: http://httpbin.org:80/get
        filters:
        - AddRequestHeader=X-Request-Foo, Bar
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
  profiles: add_request_header_route
```

在上述的配置中，工程的启动端口为8081，配置文件为add\_request\_header\_route，在add\_request\_header\_route配置中，配置了routes的id为add\_request\_header\_route，路由地址为[http://httpbin.org:80/get，该router有AfterPredictFactory，有一个filter为AddRequestHeaderGatewayFilterFactory\(约定写成AddRequestHeader\)，AddRequestHeader过滤器工厂会在请求头加上一对请求头，名称为X-Request-Foo，值为Bar。为了验证AddRequestHeaderGatewayFilterFactory是怎么样工作的，查看它的源码，AddRequestHeaderGatewayFilterFactory的源码如下：](http://httpbin.org:80/get，该router有AfterPredictFactory，有一个filter为AddRequestHeaderGatewayFilterFactory%28约定写成AddRequestHeader%29，AddRequestHeader过滤器工厂会在请求头加上一对请求头，名称为X-Request-Foo，值为Bar。为了验证AddRequestHeaderGatewayFilterFactory是怎么样工作的，查看它的源码，AddRequestHeaderGatewayFilterFactory的源码如下：)

```
public class AddRequestHeaderGatewayFilterFactory extends AbstractNameValueGatewayFilterFactory {

    @Override
    public GatewayFilter apply(NameValueConfig config) {
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest().mutate()
                    .header(config.getName(), config.getValue())
                    .build();

            return chain.filter(exchange.mutate().request(request).build());
        };
    }

}
```

由上面的代码可知，根据旧的ServerHttpRequest创建新的 ServerHttpRequest ，在新的ServerHttpRequest加了一个请求头，然后创建新的 ServerWebExchange ，提交过滤器链继续过滤。

启动工程，通过curl命令来模拟请求：

```
curl localhost:8081
```

最终显示了从 [http://httpbin.org:80/get得到了请求，响应如下：](http://httpbin.org:80/get得到了请求，响应如下：)

```
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Connection": "close",
    "Forwarded": "proto=http;host=\"localhost:8081\";for=\"0:0:0:0:0:0:0:1:56248\"",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.58.0",
    "X-Forwarded-Host": "localhost:8081",
    "X-Request-Foo": "Bar"
  },
  "origin": "0:0:0:0:0:0:0:1, 210.22.21.66",
  "url": "http://localhost:8081/get"
}
```

可以上面的响应可知，确实在请求头中加入了X-Request-Foo这样的一个请求头，在配置文件中配置的AddRequestHeader过滤器工厂生效。

跟AddRequestHeader过滤器工厂类似的还有AddResponseHeader过滤器工厂，在此就不再重复。

### RewritePath GatewayFilter Factory {#rewritepath-gatewayfilter-factory}

在Nginx服务启中有一个非常强大的功能就是重写路径，Spring Cloud Gateway默认也提供了这样的功能，这个功能是Zuul没有的。在配置文件中加上以下的配置：

```
spring:
  profiles:
    active: rewritepath_route
---
spring:
  cloud:
    gateway:
      routes:
      - id: rewritepath_route
        uri: https://blog.csdn.net
        predicates:
        - Path=/foo/**
        filters:
        - RewritePath=/foo/(?<segment>.*), /$\{segment}
  profiles: rewritepath_route
```

**上面的配置中，所有的/foo/\*\*开始的路径都会命中配置的router，并执行过滤器的逻辑**，

在本案例中配置了RewritePath过滤器工厂，此工厂将/foo/\(?.\*\)重写为{segment}，然后转发到[https://blog.csdn.net。](https://blog.csdn.net。比如在网页上请求localhost:8081/foo/forezp，此时会将请求转发到https://blog.csdn.net/forezp的页面，比如在网页上请求localhost:8081/foo/forezp/1，页面显示404，就是因为不存在https://blog.csdn.net/forezp/1这个页面。)

[比如在网页上请求localhost:8081/foo/forezp，此时会将请求转发到https://blog.csdn.net/forezp的页面，](https://blog.csdn.net。比如在网页上请求localhost:8081/foo/forezp，此时会将请求转发到https://blog.csdn.net/forezp的页面，比如在网页上请求localhost:8081/foo/forezp/1，页面显示404，就是因为不存在https://blog.csdn.net/forezp/1这个页面。)

[比如在网页上请求localhost:8081/foo/forezp/1，页面显示404，就是因为不存在https://blog.csdn.net/forezp/1这个页面。](https://blog.csdn.net。比如在网页上请求localhost:8081/foo/forezp，此时会将请求转发到https://blog.csdn.net/forezp的页面，比如在网页上请求localhost:8081/foo/forezp/1，页面显示404，就是因为不存在https://blog.csdn.net/forezp/1这个页面。)

## 自定义过滤器（自定义GatewayFilter） {#自定义过滤器}

Spring Cloud Gateway内置了19种强大的过滤器工厂，能够满足很多场景的需求，那么能不能自定义自己的过滤器呢，当然是可以的。在spring Cloud Gateway中，过滤器需要实现GatewayFilter和Ordered2个接口。写一个RequestTimeFilter，代码如下：

```
public class RequestTimeFilter implements GatewayFilter, Ordered {

    private static final Log log = LogFactory.getLog(GatewayFilter.class);
    private static final String REQUEST_TIME_BEGIN = "requestTimeBegin";

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        exchange.getAttributes().put(REQUEST_TIME_BEGIN, System.currentTimeMillis());
        return chain.filter(exchange).then(
                Mono.fromRunnable(() -> {
                    Long startTime = exchange.getAttribute(REQUEST_TIME_BEGIN);
                    if (startTime != null) {
                        log.info(exchange.getRequest().getURI().getRawPath() + ": " + (System.currentTimeMillis() - startTime) + "ms");
                    }
                })
        );

    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

在上面的代码中，Ordered中的int getOrder\(\)方法是来给过滤器设定优先级别的，值越大则优先级越低。还有有一个filterI\(exchange,chain\)方法，在该方法中，先记录了请求的开始时间，并保存在ServerWebExchange中，此处是一个“pre”类型的过滤器，然后再chain.filter的内部类中的run\(\)方法中相当于”post”过滤器，在此处打印了请求所消耗的时间。然后将该过滤器注册到router中，代码如下：

```
 @Bean
    public RouteLocator customerRouteLocator(RouteLocatorBuilder builder) {
        // @formatter:off
        return builder.routes()
                .route(r -> r.path("/customer/**")
                        .filters(f -> f.filter(new RequestTimeFilter())
                                .addResponseHeader("X-Response-Default-Foo", "Default-Bar"))
                        .uri("http://httpbin.org:80/get")
                        .order(0)
                        .id("customer_filter_router")
                )
                .build();
        // @formatter:on
    }
```

重启程序，通过curl命令模拟请求：

```
curl localhost:8081/customer/123
```

在程序的控制台输出一下的请求信息的日志：

```
2018-11-16 15:02:20.177  INFO 20488 --- [ctor-http-nio-3] o.s.cloud.gateway.filter.GatewayFilter   : /customer/123: 152ms
```

## 自定义过滤器工厂\(GatewayFilterFactory\) {#自定义过滤器工厂}

在上面的自定义过滤器中，有没有办法自定义过滤器工厂类呢?这样就可以在配置文件中配置过滤器了。现在需要实现一个过滤器工厂，在打印时间的时候，可以设置参数来决定是否打印请参数。查看GatewayFilterFactory的源码，可以发现GatewayFilterfactory的层级如下：

![img](/static/image/2279594-f97e1045cebc954c.png)



过滤器工厂的顶级接口是GatewayFilterFactory，我们可以直接继承它的两个抽象类来简化开发AbstractGatewayFilterFactory和AbstractNameValueGatewayFilterFactory，这两个抽象类的区别就是前者接收一个参数（像StripPrefix和我们创建的这种），后者接收两个参数（像AddResponseHeader）。

过滤器工厂的顶级接口是GatewayFilterFactory，有2个两个较接近具体实现的抽象类，分别为AbstractGatewayFilterFactory和AbstractNameValueGatewayFilterFactory，这2个类前者接收一个参数，比如它的实现类RedirectToGatewayFilterFactory；后者接收2个参数，比如它的实现类AddRequestHeaderGatewayFilterFactory类。现在需要将请求的日志打印出来，需要使用一个参数，这时可以参照RedirectToGatewayFilterFactory的写法。


# 参考

[https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.M1/single/spring-cloud-gateway.html](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.M1/single/spring-cloud-gateway.html)

[https://www.jianshu.com/p/eb3a67291050](https://www.jianshu.com/p/eb3a67291050)

[https://blog.csdn.net/qq\_36236890/article/details/80822051](https://blog.csdn.net/qq_36236890/article/details/80822051)

[https://windmt.com/2018/05/08/spring-cloud-14-spring-cloud-gateway-filter](https://windmt.com/2018/05/08/spring-cloud-14-spring-cloud-gateway-filter)


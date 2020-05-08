# 1.基本介绍

微服务架构中，根据业务来拆分成一个个的服务，服务与服务之间可以相互调用（RPC），在Spring Cloud可以用RestTemplate+Ribbon和Feign来调用。为了保证其高可用，单个服务通常会集群部署。由于网络原因或者自身的原因，服务并不能保证100%可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，此时若有大量的请求涌入，Servlet容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。

为了解决这个问题，业界提出了断路器模型。

Netflix开源了Hystrix组件，实现了断路器模式，SpringCloud对这一组件进行了整合。 在微服务架构中，一个请求需要调用多个服务是非常常见的，如下图：

![](/assets/微信截图_20190801180422.png)

较底层的服务如果出现故障，会导致连锁故障。当对特定的服务的调用的不可用达到一个阀值（Hystric 是5秒20次） 断路器将会被打开。

![](/assets/微信截图_20190801180735.png)

断路打开后，可用避免连锁故障，fallback方法可以直接返回一个固定值。

## 1.1.Hystrix的设计原则

原则如下：

* 防止单个服务的故障，耗尽整个系统服务的容器（比如tomcat）的线程资源。
* 减少负载并快速失败，而不是排队。
* 在可行的情况下提供回退以保护用户免受故障。
* 使用隔离技术（如隔板，泳道和断路器模式）来限制任何一个依赖的影响。
* 通过近乎实时的指标，监控和警报来优化发现故障的时间。
* 通过配置更改的低延迟传播优化恢复时间，并支持Hystrix大多数方面的动态属性更改，从而允许您使用低延迟反馈循环进行实时操作修改。
* 保护整个依赖客户端执行中的故障，而不仅仅是在网络流量上进行保护降级、限流。

## 1.2.Hystrix 是怎么实现它的设计目标的？ {#hystrix-是怎么实现它的设计目标的}

通过HystrixCommand 或者HystrixObservableCommand 将所有的外部系统（或者称为依赖）包装起来，整个包装对象是单独运行在一个线程之中（这是典型的命令模式）。

* 超时请求应该超过你定义的阈值
* 为每个依赖关系维护一个小的线程池（或信号量）; 如果它变满了，那么依赖关系的请求将立即被拒绝，而不是排队等待。
* 统计成功，失败（由客户端抛出的异常），超时和线程拒绝。
* 打开断路器可以在一段时间内停止对特定服务的所有请求，如果服务的错误百分比通过阈值，手动或自动的关闭断路器。
* 当请求被拒绝、连接超时或者断路器打开，直接执行fallback逻辑。
* 近乎实时监控指标和配置变化。

当您使用Hystrix包装每个底层依赖项时，上图所示的体系结构如下图所示。 每个依赖关系彼此隔离，在延迟发生时可以饱和的资源受到限制，迅速执行fallback的逻辑，该逻辑决定了在依赖关系中发生任何类型的故障时会做出什么响应：

soa-4-isolation-640.png

# 2.怎么使用

## 2.1.在Ribbon使用断路器

**不常用**

## 2.2.Feign中使用断路器

Feign是自带断路器的，在D版本的Spring Cloud之后，它没有默认打开。需要在配置文件中配置打开它，在配置文件加以下代码：

```
feign.hystrix.enabled=true
```

基于service-feign工程进行改造，只需要在FeignClient的SchedualServiceHi接口的注解中加上fallback的指定类就行了：

```
@FeignClient(value = "service-hi",fallback = SchedualServiceHiHystric.class)
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

SchedualServiceHiHystric需要实现SchedualServiceHi 接口，并注入到Ioc容器中，代码如下：

```
@Component
public class SchedualServiceHiHystric implements SchedualServiceHi {
    @Override
    public String sayHiFromClientOne(String name) {
        return "sorry "+name;
    }
}
```

启动四servcie-feign工程，浏览器打开[http://localhost:8765/hi?name=forezp,注意此时service-hi工程没有启动，网页显示：](http://localhost:8765/hi?name=forezp,注意此时service-hi工程没有启动，网页显示：)

```
sorry forezp
```

打开service-hi工程，再次访问，浏览器显示：

```
hi forezp,i am from port:8762
```

# 3.参考

断路器（Hystrix）\(Finchley版本\)：

[https://blog.csdn.net/forezp/article/details/81040990](https://blog.csdn.net/forezp/article/details/81040990)

Hystrix文档翻译：

[https://www.fangzhipeng.com/springcloud/2018/08/31/trans-hystrix1.html](https://www.fangzhipeng.com/springcloud/2018/08/31/trans-hystrix1.html)


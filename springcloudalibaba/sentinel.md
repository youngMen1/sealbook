# 1.什么是Sentinel {#什么是sentinel}

Sentinel，中文翻译为哨兵，是为微服务提供流量控制、熔断降级的功能，它和Hystrix提供的功能一样，可以有效的解决微服务调用产生的“雪崩”效应，为微服务系统提供了稳定性的解决方案。随着Hytrxi进入了维护期，不再提供新功能，Sentinel是一个不错的替代方案。通常情况，Hystrix采用线程池对服务的调用进行隔离，Sentinel才用了用户线程对接口进行隔离，二者相比，Hystrxi是服务级别的隔离，Sentinel提供了接口级别的隔离，Sentinel隔离级别更加精细，另外Sentinel直接使用用户线程进行限制，相比Hystrix的线程池隔离，减少了线程切换的开销。另外Sentinel的DashBoard提供了在线更改限流规则的配置，也更加的优化。

从官方文档的介绍，Sentinel 具有以下特征:

* 丰富的应用场景： Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、实时熔断下游不可用应用等。
* 完备的实时监控： Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
* 广泛的开源生态： Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
* 完善的 SPI 扩展点： Sentinel 提供简单易用、完善的 SPI 扩展点。您可以通过实现扩展点，快速的定制逻辑。例如定制规则管理、适配数据源等。

# 2.如何在Spring Cloud中使用Sentinel {#如何在spring-cloud中使用sentinel}

Sentinel作为Spring Cloud Alibaba的组件之一，在Spring Cloud项目中使用它非常的简单。现在以案例的形式来讲解如何在Spring Cloud项目中使用Sentinel。本项目是在之前nacos教程的案例基础上进行改造。在工程的pom文件加上sentinel的Spring Cloud起步依赖，代码如下：

```
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
     <version>0.9.0.RELEASE</version> 
</dependency>
```

在工程的配置文件application.yml文件中配置，需要新增2个配置：

* spring.cloud.sentinel.transport.port: 8719 ，这个端口配置会在应用对应的机器上启动一个 Http Server，该 Server 会与 Sentinel 控制台做交互。比如 Sentinel 控制台添加了1个限流规则，会把规则数据 push 给这个 Http Server 接收，Http Server 再将规则注册到 Sentinel 中。
* spring.cloud.sentinel.transport.dashboard: 8080，这个是Sentinel DashBoard的地址。

```
server:
  port: 8762
spring:
  application:
    name: nacos-provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    sentinel:
      transport:
        port: 8719
        dashboard: localhost:8080
```

写一个RestController，在接口上加上SentinelResource注解就可以了。

```
@RestController
public class ProviderController {

    @GetMapping("/hi")
    @SentinelResource(value="hi")
    public String hi(@RequestParam(value = "name",defaultValue = "forezp",required = false)String name){

        return "hi "+name;
    }

}
```

关于@SentinelResource 注解，有以下的属性：

* value：资源名称，必需项（不能为空）
* entryType：entry 类型，可选项（默认为 EntryType.OUT）
* blockHandler / blockHandlerClass: blockHandler 对应处理 BlockException 的函数名称，可选项
* fallback：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。

启动Nacos，并启动nacos-provider项目。文末有源码下载链接。

## 2.1.Sentinel DashBoard {#sentinel-dashboard}

Sentinel 控制台提供一个轻量级的控制台，它提供机器发现、单机资源实时监控、集群资源汇总，以及规则管理的功能. Sentinel DashBoard下载地址：[https://github.com/alibaba/Sentinel/releases](https://github.com/alibaba/Sentinel/releases)

下载完成后，以以下的命令启动

```
java -jar sentinel-dashboard-1.6.1.jar
```

默认启动端口为8080，可以-Dserver.port=8081的形式改变默认端口。启动成功后，在浏览器上访问localhost:8080，就可以显示Sentinel的登陆界面，登陆名为sentinel，密码为sentinel。

登陆sentinel dashboard成功后，并多次访问nacos-provider的localhost:8080/hi接口，在nacos访问信息如下：

![](https://www.fangzhipeng.com/img/jianshu/2279594-51bbd0f299471d4d.png "WX20190602-193550@2x.png")

sentinel dashboard显示了nacos-provider的接口资源信息。

![](https://www.fangzhipeng.com/img/jianshu/2279594-0ceba359c2e1daff.png "WX20190602-191843@2x.png")

在/hi资源处设置接口的限流功能，在“+流控”按钮点击开设置界面如下,设置阈值类型为 qps，单机阈值为2。

![](https://www.fangzhipeng.com/img/jianshu/2279594-ac4e0be06515ec9a.png "WX20190602-191900@2x.png")

设置成功后可以在流控规则这一栏进行查看，如图所示：

![](https://www.fangzhipeng.com/img/jianshu/2279594-367002bee1cc0232.png "WX20190602-191931@2x.png")

### 测试 {#测试}

多次快速访问nacos-provider的接口资源[http://localhost:8762/hi，可以发现偶尔出现以下的信息：](http://localhost:8762/hi，可以发现偶尔出现以下的信息：)

> Blocked by Sentinel \(flow limiting\)

正常的返回逻辑为

> hi forezp

由以上可只，接口资源/hi的限流规则起到了作用。

## 2.2.在FeignClient中使用Sentinel {#在feignclient中使用sentinel}

Hystrix默认集成在Spring Cloud 的Feign Client组件中，Sentinel也可以提供这样的功能。现以案例的形式来讲解如何在FeignClient中使用Sentinel，z本案例是在之前的nacos教程案例的nacos-consumer工程上进行改造，除了引入spring-cloud-starter-alibaba-sentinel，还需要引入spring-cloud-starter-openfeign，代码如下：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    <version>0.9.0.RELEASE</version>
</dependency>
```

在配置文件中需要加上sentinel.transport. dashboard配置外，还需要加上feign.sentinel.enabled的配置，代码如下：

```
server:
  port: 8763
spring:
  application:
    name: nacos-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    sentinel:
      transport:
        port: 8719
        dashboard: localhost:8080

feign.sentinel.enabled: true
```

写一个FeignClient，调用nacos-provider的/hi接口：

```
@FeignClient("nacos-provider")
public interface ProviderClient {

    @GetMapping("/hi")
    String hi(@RequestParam(value = "name", defaultValue = "forezp", required = false) String name);
}
```

写一个RestController调用ProviderClient，代码如下：

```
@RestController
public class ConsumerController {

    @Autowired
    ProviderClient providerClient;

    @GetMapping("/hi-feign")
    public String hiFeign(){
       return providerClient.hi("feign");
    }
}
```

在FeignClient中，Sentinel为Feign调用生成了资源名策略定义，定义规则为httpmethod:protocol://requesturl。启动nacos-consumer工程，在Sentinel DashBoard生成了如下的资源信息：

![](https://www.fangzhipeng.com/img/jianshu/2279594-1ae0b0f5436bb9b5.png "WX20190602-195133@2x.png")

添加流控，QPS为2，在浏览器上快速多次点击访问[http://localhost:8763/hi-feign，浏览器在正常情况下是能够正常返回如下的信息：](http://localhost:8763/hi-feign，浏览器在正常情况下是能够正常返回如下的信息：)

> hi feign

在被限流的时候返回错误信息。

需要注意的是，被限流的时候FeignClient并不会调用nacos-provider的接口，而是在nacos-consumer工程里直接报错。

# 3.总结

源码地址:[https://github.com/youngMen1/springcloud-alibaba-code](https://github.com/youngMen1/springcloud-alibaba-code)

[https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-discovery](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-discovery)

# 4.参考

https://www.fangzhipeng.com/springcloud/2019/06/02/sc-sentinel.html




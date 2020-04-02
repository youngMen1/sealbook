# 1.基本介绍

#### Sentinel，中文翻译为哨兵，是为微服务提供流量控制、熔断降级的功能，它和Hystrix提供的功能一样，可以有效的解决微服务调用产生的“雪崩”效应，为微服务系统提供了稳定性的解决方案。随着Hytrxi进入了维护期，不再提供新功能，Sentinel是一个不错的替代方案。通常情况，Hystrix采用线程池对服务的调用进行隔离，Sentinel才用了用户线程对接口进行隔离，二者相比，Hystrxi是服务级别的隔离，Sentinel提供了接口级别的隔离，Sentinel隔离级别更加精细，另外Sentinel直接使用用户线程进行限制，相比Hystrix的线程池隔离，减少了线程切换的开销。另外Sentinel的DashBoard提供了在线更改限流规则的配置，也更加的优化。

Sentinel 具有以下特征:

* 丰富的应用场景： Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、实时熔断下游不可用应用等。
* 完备的实时监控： Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
* 广泛的开源生态： Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
* 完善的 SPI 扩展点： Sentinel 提供简单易用、完善的 SPI 扩展点。您可以通过实现扩展点，快速的定制逻辑。例如定制规则管理、适配数据源等。

# 2.如何在Spring Cloud中使用Sentinel

Sentinel作为Spring Cloud Alibaba的组件之一，在Spring Cloud项目中使用它非常的简单,在工程的pom文件加上sentinel的Spring Cloud起步依赖，代码如下：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-alibaba-nacos-config</artifactId>
    <version>0.9.0.RELEASE</version>
</dependency>
```

在工程的配置文件application.yml文件中配置，需要新增2个配置：

* spring.cloud.sentinel.transport.port: 8719 ，这个端口配置会在应用对应的机器上启动一个 Http Server，该 Server 会与 Sentinel 控制台做交互。比如 Sentinel 控制台添加了1个限流规则，会把规则数据 push 给这个 Http Server 接收，Http Server 再将规则注册到 Sentinel 中。
* spring.cloud.sentinel.transport.dashboard: 8080，这个是Sentinel DashBoard的地址。

```
server:
  port: 8765
spring:
  application:
    name: nacos-sentinel
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        port: 8719
        dashboard: localhost:8080
```

写一个TestController，在接口上加上SentinelResource注解就可以了:

```
@RestController
public class TestController {

    @GetMapping("/hi")
    @SentinelResource(value = "hi")
    public String hi(@RequestParam(value = "name", defaultValue = "forezp", required = false) String name) {
        return "hi " + name;
    }
}
```

关于@SentinelResource 注解，有以下的属性：

* value：资源名称，必需项（不能为空）
* entryType：entry 类型，可选项（默认为 EntryType.OUT）
* blockHandler / blockHandlerClass: blockHandler 对应处理 BlockException 的函数名称，可选项
* fallback：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。
* 启动Nacos，并启动nacos-provider项目。文末有源码下载链接。

# 3.总结

源码地址:

# 4.参考




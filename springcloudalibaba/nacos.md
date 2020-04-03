# 1.Nacos是什么

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。 是Spring Cloud A 中的服务注册发现组件，类似于Consul、Eureka，同时它又提供了分布式配置中心的功能，这点和Consul的config类似，支持热加载。  
Nacos 的关键特性包括:

* 服务发现和服务健康监测
* 动态配置服务，带管理界面，支持丰富的配置维度。
* 动态 DNS 服务
* 服务及其元数据管理

# 2.使用Nacos服务注册和发现

服务注册和发现是微服务治理的根基，服务注册和发现组件是整个微服务系统的灵魂，选择合适的服务注册和发现组件至关重要，目前主流的服务注册和发现组件有Consul、Eureka、Etcd等。 随着Eureka的闭源，Spring cloud netflix-oss组件大规模的进入到了维护期，不再提供新功能，spring cloud alibaba受到开源社区的大力拥护。

## 2.1.Nacos下载

Nacos依赖于Java环境，所以必须安装Java环境。然后从官网下载Nacos的解压包，安装稳定版的，下载地址：

```
https://github.com/alibaba/nacos/releases
```

本次案例下载的版本为1.0.0 ，下载完成后，解压，在解压后的文件的/bin目录下:

* windows系统点击startup.cmd就可以启动nacos。
* linux或mac执行以下命令启动nacos。

启动成功，在浏览器上访问：[http://localhost:8848/nacos，会跳转到登陆界面，默认的登陆用户名为nacos，密码也为nacos。](http://localhost:8848/nacos，会跳转到登陆界面，默认的登陆用户名为nacos，密码也为nacos。)

启动成功界面:

![img](/static/image/微信截图_20200402170324.png)  
从界面可知，此时没有服务注册到Nacos上。

## 2.2.服务注册

在本案例中，使用2个服务注册到Nacos上，分别为nacos-provider和nacos-consumer。

## 2.3.构建服务提供者nacos-provider

Spring boot版本为2.1.4.RELEASE，Spring Cloud 版本为Greenwich.RELEASE，在pom文件引入nacos的Spring Cloud起步依赖，代码如下：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>0.9.0.RELEASE</version>
</dependency>
```

application.yml做相关的配置如下:

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
```

在Spring Boot的启动文件NacosProviderApplication加上@EnableDiscoveryClient注解：

```
@EnableDiscoveryClient
@SpringBootApplication
public class SpringcloudNacosProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudNacosProviderApplication.class, args);
    }

}
```

## 2.4.构建服务消费者nacos-consumer

和nacos-provider一样，构建服务消费者springcloud-nacos-consumer，springcloud-nacos-cosumer的启动端口8763。构建过程同nacos-provider一样,这里省略......

## 2.5.验证服务注册个发现

分别启动2个工程，待工程启动成功之后，在访问localhost:8848，可以发现nacos-provider和nacos-consumer，均已经向nacos-server注册，如下图所示：  
![img](/static/image/微信截图_20200402172329.png)  
![img](/static/image/微信截图_20200402172301.png)

## 2.6.FeignClient调用服务 {#是feignclient调用服务}

在nacos-consumer的pom文件引入以下的依赖：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

在NacosConsumerApplication启动文件上加上@EnableFeignClients注解开启FeignClient的功能

```
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class SpringcloudNacosConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudNacosConsumerApplication.class, args);
    }

}
```

写一个FeignClient，调用nacos-provider的服务，代码如下：

```
@FeignClient("nacos-provider")
public interface ProviderClient {

    @GetMapping("/hi")
    String hi(@RequestParam(value = "name", defaultValue = "fengzhiqiang", required = false) String name);
}
```

在nacos-consumer写一个消费API，该API使用ProviderClient来调用nacos-provider的API服务，代码如下：

```
@RestController
public class TestController {

    @Autowired
    ProviderClient providerClient;

    @GetMapping("/hi-feign")
    public String hiFeign() {
        return providerClient.hi("fengzhiqiang");
    }
}
```

在浏览器上访问[http://localhost:8763/hi-feign，可以在浏览器上展示正确的响应，这时nacos-consumer调用nacos-provider服务成功。](http://localhost:8763/hi-feign，可以在浏览器上展示正确的响应，这时nacos-consumer调用nacos-provider服务成功。)  
![img](/static/image/微信截图_20200402173226.png)  
![img](/static/image/微信截图_20200403085716.png)

# 3.如何使用nacos作为配置中心

nacos作为服务配置中心的功能。类似于consul config，Nacos 是支持热加载的。

springcloud-nacos-config工程上改造的，在工程的pom文件引入nacos-config的Spring cloud依赖，版本为0.9.0. RELEASE，代码如下：

```

```

# 4.总结

Nacos下载地址:[https://github.com/alibaba/nacos/releases](https://github.com/alibaba/nacos/releases)

# 5.参考

[https://nacos.io/zh-cn/docs/what-is-nacos.html](https://nacos.io/zh-cn/docs/what-is-nacos.html)

[https://www.fangzhipeng.com/springcloud/2019/06/01/sc-nacos-config.html](https://www.fangzhipeng.com/springcloud/2019/06/01/sc-nacos-config.html)


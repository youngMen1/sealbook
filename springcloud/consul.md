# 1.Consul是什么

Consul是HashiCorp公司推出的开源软件，使用GO语言编写，提供了**分布式系统的服务注册和发现、配置等功能**，这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格。Consul不仅具有服务治理的功能，而且使用分布式一致协议RAFT算法实现，有多数据中心的高可用方案，并且很容易和Spring Cloud等微服务框架集成，使用起来非常的简单，具有简单、易用、可插排等特点。使用简而言之，Consul提供了一种完整的服务网格解决方案。

## 1.1.Consul具有以下的特点和功能

* 服务发现：Consul的客户端可以向Consul注册服务，例如api服务或者mysql服务，其他客户端可以使用Consul来发现服务的提供者。Consul支持使用DNS或HTTP来注册和发现服务。
* 运行时健康检查：Consul客户端可以提供任意数量的运行状况检查机制，这些检查机制可以是给定服务（“是Web服务器返回200 OK”）或本地节点（“内存利用率低于90％”）相关联。这些信息可以用来监控群集的运行状况，服务发现组件可以使用这些监控信息来路由流量，可以使流量远离不健康的服务。
* KV存储：应用程序可以将Consul的键/值存储用于任何需求，包括动态配置，功能标记，协调，领导者选举等。它采用HTTP API使其易于使用。
* 安全服务通信：Consul可以为服务生成和分发TLS证书，以建立相互的TLS连接。
* 多数据中心：Consul支持多个数据中心。这意味着Consul的用户不必担心构建额外的抽象层以扩展到多个区域。

## 1.2.Consul原理

每个提供服务的节点都运行了Consul的代理，运行代理不需要服务发现和获取配置的KV键值对，代理只负责监控检查。代理节点可以和一个或者多个Consul server通讯。 Consul服务器是存储和复制数据的地方。服务器本身选出了领导者。虽然Consul可以在一台服务器上运行，但建议使用3到5，以避免导致数据丢失的故障情况。建议为每个数据中心使用一组Consul服务器。 如果你的组件需要发现服务，可以查询任何Consul Server或任何Consul客户端，Consul客户端会自动将查询转发给Consul Server。 需要发现其他服务或节点的基础架构组件可以查询任何Consul服务器或任何Consul代理。代理会自动将查询转发给服务器。每个数据中心都运行Consul服务器集群。发生跨数据中心服务发现或配置请求时，本地Consul服务器会将请求转发到远程数据中心并返回结果。

## 1.3.术语

* Agent agent是一直运行在Consul集群中每个成员上的守护进程。通过运行 consul agent 来启动。agent可以运行在client或者server模式。指定节点作为client或者server是非常简单的，除非有其他agent实例。所有的agent都能运行DNS或者HTTP接口，并负责运行时检查和保持服务同步。
* Client 一个Client是一个转发所有RPC到server的代理。这个client是相对无状态的。client唯一执行的后台活动是加入LAN gossip池。这有一个最低的资源开销并且仅消耗少量的网络带宽。
* Server 一个server是一个有一组扩展功能的代理，这些功能包括参与Raft选举，维护集群状态，响应RPC查询，与其他数据中心交互WAN gossip和转发查询给leader或者远程数据中心。
* DataCenter 虽然数据中心的定义是显而易见的，但是有一些细微的细节必须考虑。例如，在EC2中，多个可用区域被认为组成一个数据中心？我们定义数据中心为一个私有的，低延迟和高带宽的一个网络环境。这不包括访问公共网络，但是对于我们而言，同一个EC2中的多个可用区域可以被认为是一个数据中心的一部分。
* Consensus 在我们的文档中，我们使用Consensus来表明就leader选举和事务的顺序达成一致。由于这些事务都被应用到有限状态机上，Consensus暗示复制状态机的一致性。
* Gossip Consul建立在Serf的基础之上，它提供了一个用于多播目的的完整的gossip协议。Serf提供成员关系，故障检测和事件广播。更多的信息在gossip文档中描述。这足以知道gossip使用基于UDP的随机的点到点通信。
* LAN Gossip 它包含所有位于同一个局域网或者数据中心的所有节点。
* WAN Gossip 它只包含Server。这些server主要分布在不同的数据中心并且通常通过因特网或者广域网通信。
* RPC 远程过程调用。这是一个允许client请求server的请求/响应机制。
  ![img](/static/image/consul-arch-420ce04a.png)

让我们分解这张图并描述每个部分。首先，我们能看到有两个数据中心，标记为“1”和“2”。Consul对多数据中心有一流的支持并且希望这是一个常见的情况。

在每个数据中心，client和server是混合的。一般建议有3-5台server。这是基于有故障情况下的可用性和性能之间的权衡结果，因为越多的机器加入达成共识越慢。然而，并不限制client的数量，它们可以很容易的扩展到数千或者数万台。

同一个数据中心的所有节点都必须加入gossip协议。这意味着gossip协议包含一个给定数据中心的所有节点。这服务于几个目的：

* 第一，不需要在client上配置server地址。发现都是自动完成的。
* 第二，检测节点故障的工作不是放在server上，而是分布式的。这是的故障检测相比心跳机制有更高的可扩展性。
* 第三：它用来作为一个消息层来通知事件，比如leader选举发生时。

每个数据中心的server都是Raft节点集合的一部分。这意味着它们一起工作并选出一个leader，一个有额外工作的server。leader负责处理所有的查询和事务。作为一致性协议的一部分，事务也必须被复制到所有其他的节点。因为这一要求，当一个非leader得server收到一个RPC请求时，它将请求转发给集群leader。

server节点也作为WAN gossip Pool的一部分。这个Pool不同于LAN Pool，因为它是为了优化互联网更高的延迟，并且它只包含其他Consul server节点。这个Pool的目的是为了允许数据中心能够以low-touch的方式发现彼此。这使得一个新的数据中心可以很容易的加入现存的WAN gossip。因为server都运行在这个pool中，它也支持跨数据中心请求。当一个server收到来自另一个数据中心的请求时，它随即转发给正确数据中想一个server。该server再转发给本地leader。

这使得数据中心之间只有一个很低的耦合，但是由于故障检测，连接缓存和复用，跨数据中心的请求都是相对快速和可靠的。

## 1.4.Consul 服务注册发现流程

Consul在业界最广泛的用途就是作为服务注册中心，同Eureka类型，consul作为服务注册中心，它的注册和发现过程如下图：  
![img](/static/image/2279594-89ad0386fbbb93e3.png)  
在上面的流程图上有三个角色，分别为服务注册中心、服务提供者、服务消费者。

* 服务提供者Provider启动的时候，会向Consul发送一个请求，将自己的host、ip、应用名、健康检查等元数据信息发送给Consul
* Consul 接收到 Provider 的注册后，定期向 Provider 发送健康检查的请求，检验Provider是否健康
* 服务消费者Consumer会从注册中心Consul中获取服务注册列表，当服务消费者消费服务时，根据应用名从服务注册列表获取到具体服务的实例（1个或者多个），从而完成服务的调用。

## 1.5.Consul VS Eureka

Eureka是一种服务发现工具。 该体系结构主要是客户端/服务器，每个数据中心有一组Eureka服务器，通常每个可用区域一个。 通常，Eureka的客户使用嵌入式SDK来注册和发现服务。 对于非本地集成的客户端，使用Ribbon等边车通过Eureka透明地发现服务。

Eureka使用尽力而为的复制提供弱一致的服务视图。 当客户端向服务器注册时，该服务器将尝试复制到其他服务器但不提供保证。 服务注册的生存时间很短（TTL），要求客户端对服务器进行心跳检测。 不健康的服务或节点将停止心跳，导致它们超时并从注册表中删除。 发现请求可以路由到任何服务，由于尽力复制，这些服务可以提供过时或丢失的数据。 这种简化的模型允许轻松的集群管理和高可扩展性。

Consul提供了一系列超级功能，包括更丰富的运行状况检查，键/值存储和多数据中心感知。 Consul需要每个数据中心中的一组服务器，以及每个客户端上的代理，类似于使用像Ribbon这样的边车。 Consul代理允许大多数应用程序不知道Consul，通过配置文件执行服务注册以及通过DNS或负载平衡器sidecars进行发现。

Consul提供强大的一致性保证，因为服务器使用Raft协议复制状态。 Consul支持丰富的运行状况检查，包括TCP，HTTP，Nagios / Sensu兼容脚本或基于的Eureka的TTL。 客户端节点参与基于gossip的健康检查，该检查分发健康检查的工作，而不像集中式心跳，这成为可扩展性挑战。 发现请求被路由到当选的Consul领导者，这使他们默认情况下非常一致。 允许过时读取的客户端允许任何服务器处理其请求，从而允许像Eureka一样的线性可伸缩性。

Consul的强烈一致性意味着它可以用作领导者选举和集群协调的锁定服务。 Eureka不提供类似的保证，并且通常需要为需要执行协调或具有更强一致性需求的服务运行ZooKeeper。

Consul提供了支持面向服务的体系结构所需的功能工具包。 这包括服务发现，还包括丰富的运行状况检查，锁定，键/值，多数据中心联合，事件系统和ACL。 Consul和consul-template和envconsul等工具生态系统都试图最大限度地减少集成所需的应用程序更改，以避免需要通过SDK进行本机集成。 Eureka是更大的Netflix OSS套件的一部分，该套件期望应用程序相对同质且紧密集成。 因此，Eureka只解决了有限的一部分问题，期望其他工具如ZooKeeper可以同时使用。

Eureka Server端采用的是P2P的复制模式，但是它不保证复制操作一定能成功，因此它提供的是一个最终一致性的服务实例视图；Client端在Server端的注册信息有一个带期限的租约，一旦Server端在指定期间没有收到Client端发送的心跳，则Server端会认定为Client端注册的服务是不健康的，定时任务将会将其从注册表中删除。Consul与Eureka不同，Consul采用Raft算法，可以提供强一致性的保证，Consul的agent相当于Netflix Ribbon + Netflix Eureka Client，而且对应用来说相对透明，同时相对于Eureka这种集中式的心跳检测机制，Consul的agent可以参与到基于goosip协议的健康检查，分散了server端的心跳检测压力。除此之外，Consul为多数据中心提供了开箱即用的原生支持等。

# 2.怎么使用Consul

## 2.1.Consul下载和安装

```
Consul采用Go语言编写，支持Linux、Mac、Windows等各大操作系统，本文使用windows操作系统，下载地址：https://www.consul.io/downloads.html，下完成后解压到计算机目录下，解压成功后，只有一个可执行的consul.exe可执行文件。打开cmd终端，切换到目录，执行以下命令：
```

终端显示如下：

```
Consul v1.4.2
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use p
rotocol >2 when speaking to compatible agents)
```

证明consul下载成功了，并可执行。

consul的一些常见的执行命令如下：

| 命令 | 解释 | 示例 |
| :--- | :--- | :--- |
| agent | 运行一个consul agent | consul agent -dev |
| join | 将agent加入到consul集群 | consul join IP |
| members | 列出consul cluster集群中的members | consul members |
| leave | 将节点移除所在集群 | consul leave |

更多命令请查看官方网站：[https://www.consul.io/docs/commands/index.html](https://www.consul.io/docs/commands/index.html)

开发模式启动：

```
consul agent -dev
```

启动成功，在浏览器上访问：[http://localhost:8500，显示的界面如下：](http://localhost:8500，显示的界面如下：)

![img](/static/image/微信截图_20200403135940.png)

## 2.2.spring cloud consul

该项目通过自动配置并绑定到Spring环境和其他Spring编程模型成语，为Spring Boot应用程序提供Consul集成。通过几个简单的注释，您可以快速启用和配置应用程序中的常见模式，并使用基于Consul的组件构建大型分布式系统。提供的模式包括服务发现，控制总线和配置。智能路由（Zuul）和客户端负载平衡（Ribbon），断路器（Hystrix）通过与Spring Cloud Netflix的集成提供。

## 2.2.使用spring cloud consul来服务注册与发现

本小节以案例的形式来讲解如何使用Spring Cloud Consul来进行服务注册和发现的，并且使用Feign来消费服务。再讲解之前，已经启动consul的agent，并且在浏览器上[http://localhost:8500能够显示正确的页面。本案例一共有2个工程，分别如下：](http://localhost:8500能够显示正确的页面。本案例一共有2个工程，分别如下：)

| 工程名 | 端口 | 描述 |
| :--- | :--- | :--- |
| springcloud-consul-provider | 8763 | 服务提供者 |
| springcloud-consul-consumer | 8765 | 服务消费者 |

其中，服务提供者和服务消费者分别向consul注册，注册完成后，服务消费者通过FeignClient来消费服务提供者的服务。

### 2.2.1服务提供者springcloud-consul-provider

1.创建一个工程springcloud-consul-provider，在工程的pom文件引入以下依赖，包括consul-discovery的起步依赖，该依赖是spring cloud consul用来向consul 注册和发现服务的依赖，采用REST API的方式进行通讯。另外加上web的起步依赖，用于对外提供REST API。代码如下：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

2.在工程的配置文件application.yml做下以下配置：

```
server:
  port: 8763
spring:
  application:
    name: consul-provider
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        serviceName: consul-provider
```

3.上面的配置，指定了程序的启动端口为8763，应用名为consul-provider，consul注册中心的地址为localhost:8500  
在程序员的启动类ConsulProviderApplication加上@EnableDiscoveryClient注解，开启服务发现的功能。

```
@EnableDiscoveryClient
@SpringBootApplication
public class SpringcloudConsulProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudConsulProviderApplication.class, args);
    }

}
```

4.写一个TestController，该API为一个GET请求，返回当前程序的启动端口，代码如下。

```
@RestController
public class TestController {

    @Value("${server.port}")
    String port;

    @GetMapping("/hi")
    public String home(@RequestParam String name) {
        return "hi " + name + ",i am from port:" + port;
    }
}
```

5.启动工程，在浏览器上访问[http://localhost:8500，页面显示如下：](http://localhost:8500，页面显示如下：)  
![img](/static/image/微信截图_20200403143808.png)  
从上图可知，consul-provider服务已经成功注册到consul上面去了。

### 2.2.2.服务消费者springcloud-consul-consumer

1.服务消费者的搭建过程同服务提供者，在pom文件中引入的依赖同服务提供者，在配置文件application.yml配置同服务提供者，不同的点在端口为8765，服务名为consul-consumer。  
引入

```
<!--通过 @EnableFeginClients 注解开启 Feign 功能-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2.配置如下

```
server:
  port: 8767
spring:
  application:
    name: consul-consumer
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        serviceName: consul-consumer
```

3.写一个FeignClient，该FeignClient调用consul-provider的REST API，代码如下：

```
@FeignClient(value = "consul-provider")
public interface EurekaClientFeign {

    @GetMapping(value = "/hi")
    String sayHiFromClientEureka(@RequestParam(value = "name") String name);
}

```

4.Service层代码如下：
```

@Service  
public class HiService {


@Autowired
EurekaClientFeign eurekaClientFeign;


public String sayHi(String name) {
    return eurekaClientFeign.sayHiFromClientEureka(name);
}
}
```
5.对外提供一个REST API，该API调用了consul-provider的服务，代码如下：
```
@RestController  
public class TestController {

@Autowired
HiService hiService;

@GetMapping("/hi")
public String sayHi(@RequestParam(defaultValue = "fengzhiqiang", required = false) String name) {
    return hiService.sayHi(name);
}
```

6.在浏览器上访问[http://localhost:8767/hi，浏览器响应如下：](http://localhost:8765/hi，浏览器响应如下：)  
![img](/static/image/微信截图_20200403152635.png)  
![img](/static/image/微信截图_20200403152001.png)

## 2.3.使用Spring Cloud Consul Config来做服务配置中心

1.Consul不仅能用来服务注册和发现，Consul而且支持Key/Value键值对的存储，可以用来做配置中心。Spring Cloud 提供了Spring Cloud Consul Config依赖去和Consul相集成，用来做配置中心。 现在以案例的形式来讲解如何使用Consul作为配置中心，本案例在上一个案例的consul-provider基础上进行改造。首先在工程的pom文件加上consul-config的起步依赖，代码如下：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-config</artifactId>
</dependency>
```

2.然后在配置文件application.yml加上以下的以下的配置，配置如下：

```
spring:
  profiles:
    active: dev
```

3.上面的配置指定了SpringBoot启动时的读取的profiles为dev。 然后再工程的启动配置文件bootstrap.yml文件中配置以下的配置：

```
spring:
  application:
    name: consul-provider
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        serviceName: consul-provider
      config:
        enabled: true
        format: yaml           
        prefix: config     
        profile-separator: ':'    
        data-key: data
```

关于spring.cloud.consul.config的配置项描述如下：

enabled 设置config是否启用，默认为true  
format 设置配置的值的格式，可以yaml和properties  
prefix 设置配的基本目录，比如config  
defaultContext 设置默认的配置，被所有的应用读取，本例子没用的  
profileSeparator profiles配置分隔符,默认为‘,’  
date-key为应用配置的key名字，值为整个应用配置的字符串。  
**网页上访问consul的KV存储的管理界面，即**[http://localhost:8500/ui/dc1/kv，创建一条记录，](http://localhost:8500/ui/dc1/kv，创建一条记录，)**  
key值为：**

```
config/consul-provider:dev/data
```

**value值如下:**

```
foo:  
  bar: bar1  
server:  
  port: 8081
```

![img](/static/image/微信截图_20200403154513.png)

在consul-provider工程新建一个API，该API返回从consul 配置中心读取foo.bar的值，代码如下：

```
@RestController  
public class FooBarController {  
    @Value\("${foo.bar}"\)  
    String fooBar;
@GetMapping("/foo")
public String getFooBar() {
    return fooBar;
}
```

4.启动工程，可以看到程序的启动端口为8081，即是consul的配置中心配置的server.port端口。 工程启动完成后，在浏览器上访问[http://localhost:8081/foo，页面显示bar1。由此可知，应用consul-provider已经成功从consul的配置中心读取了配置foo.bar的配置。](http://localhost:8081/foo，页面显示bar1。由此可知，应用consul-provider已经成功从consul的配置中心读取了配置foo.bar的配置。)

![img](/static/image/微信截图_20200403155258.png)



### 动态刷新配置 {#动态刷新配置}

当使用spring cloud config作为配置中心的时候，可以使用spring cloud config bus支持动态刷新配置。Spring Cloud Comsul Config默认就支持动态刷新，只需要在需要动态刷新的类上加上@RefreshScope注解即可，修改代码如下：

```
@RestController
@RefreshScope
public class FooBarController {

    @Value("${foo.bar}")
    String fooBar;

    @GetMapping("/foo")
    public String getFooBar() {
        return fooBar;
    }
}
```

启动consul-provider工程，在浏览器上访问http://localhost:8081/foo，页面显示bar1。然后 在网页上访问consul的KV存储的管理界面，即http://localhost:8500/ui/dc1/kv，修改config/consul-provider:dev/data的值，修改后的值如下：

```
foo:
  bar: fengzhiqiang
server: 
  port: 8081
```

![img](/static/image/微信截图_20200403155212.png)

此时不重新启动consul-provider，在浏览器上访问http://localhost:8081/foo，页面显示bar2。可见foo.bar的最新配置在应用不重启的情况下已经生效。
![img](/static/image/微信截图_20200403155626.png)

# 3.总结
**注意事项:**
* consul支持的KV存储的Value值不能超过512KB
* Consul的dev模式，所有数据都存储在内存中，重启Consul的时候会导致所有数据丢失，在正式的环境中，Consul的数据会持久化，数据不会丢失。

Consul采用Go语言编写，支持Linux、Mac、Windows等各大操作系统  
下载地址：[https://www.consul.io/downloads.html](https://www.consul.io/downloads.html)

# 4.参考
https://www.fangzhipeng.com/springcloud/2019/02/14/sc-consul-g.html
[https://www.consul.io/intro/index.html](https://www.consul.io/intro/index.html)
[https://www.consul.io/docs/internals/architecture.html](https://www.consul.io/docs/internals/architecture.html)
[https://www.consul.io/intro/vs/eureka.html](https://www.consul.io/intro/vs/eureka.html)
[http://www.ityouknow.com/springcloud/2018/07/20/spring-cloud-consul.html](http://www.ityouknow.com/springcloud/2018/07/20/spring-cloud-consul.html)
[https://springcloud.cc/spring-cloud-consul.html](https://springcloud.cc/spring-cloud-consul.html)
[https://www.cnblogs.com/lsf90/p/6021465.html](https://www.cnblogs.com/lsf90/p/6021465.html)
[https://blog.csdn.net/longgeqiaojie304/article/details/85227936](https://blog.csdn.net/longgeqiaojie304/article/details/85227936)


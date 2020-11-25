# 1.SkyWalking 分布式追踪系统

## 1.1.SkyWalking简介

```
SkyWalking是一个开源的观测平台，用于从服务和云原生等基础设施中收集、分析、聚合以及可视化数据。
SkyWalking 提供了一种简便的方式来清晰地观测分布式系统，甚至可以观测横跨不同云的系统。
SkyWalking 更像是一种现代的应用程序性能监控（Application Performance Monitoring，即APM）工具，专为云原生，
基于容器以及分布式系统而设计
```

SkyWalking 在逻辑上分为四部分：**探针、平台后端、存储和用户界面**。其架构图如下：  
![](/static/image/19037705-c6cd5fe0547e57a1.webp)

* 探针：基于不同的来源探针可能是不一样的，但作用都是收集数据，将数据格式化为 SkyWalking 适用的格式。例如在Java中则是做字节码植入，无侵入式的收集，并通过 HTTP 或者 gRPC 方式发送数据到平台后端

* 平台后端：是一个支持集群模式运行的后台，用于数据聚合、数据分析以及驱动数据流从探针到用户界面的流程。平台后端还提供了各种可插拔的能力，如不同来源数据（如来自 Zipkin）格式化，不同存储系统以及集群管理。你甚至还可以使用观测分析语言来进行自定义聚合分析。

* 存储：是开放式的，可以选择一个既有的存储系统，如 ElasticSearch、H2 或 MySQL 集群（Sharding-Sphere 管理），也可以选择自己实现一个存储系统。

* 用户界面：也就是SkyWalking的可视化界面，UI非常炫酷且强大，同样它也是可定制以匹配你已存在的后端的

SkyWalking 为观察和监控分布式系统提供了许多不同场景下的解决方案。例如为Java、C\#及Node.js提供语言自动探针，无侵入式的收集。同时也为一些编译型语言C++、GO等提供了手动打点 SDK（目前还未支持）。除此之外，还可以使用服务网格基础探针来收集数据，以帮助了解整个分布式系统。

在SkyWalking中也存在服务、服务实例及端点概念，因为SkyWalking就是提供了这些概念的观测能力：

* 服务（Service）：表示对请求提供相同行为的一系列或一组工作负载。在使用打点代理或 SDK 的时候，你可以定义服务的名字。如果不定义的话，SkyWalking 将会使用你在平台上定义的名字，如 Istio。

* 服务实例（Service Instance）：上述的一组工作负载中的每一个工作负载称为一个实例。就像 Kubernetes 中的 pods 一样，服务实例未必就是操作系统上的一个进程。但当你在使用打点代理的时候， 一个服务实例实际就是操作系统上的一个真实进程。

* 端点（Endpoint）：对于特定服务所接收的请求路径，如 HTTP 的 URI 路径和 gRPC 服务的类名 + 方法签名

综上，SkyWalking 优势如下：

* 多种监控手段，语言探针和服务网格（Service Mesh）
* 模块化，UI、存储、集群管理多种机制可选
* 支持告警
* 优秀的可视化方案

更多内容可以参考官方文档：

* [SkyWalking 文档中文版（社区提供）](https://links.jianshu.com/go?to=https%3A%2F%2Fskyapm.github.io%2Fdocument-cn-translation-of-skywalking%2F)

* [Apache SkyWalking 官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fapache%2Fskywalking%2Ftree%2Fmaster%2Fdocs)

## 1.2.Linux环境搭建SkyWalking服务

对SkyWalking有一个大致的了解后，本小节我们来在CentOS7上搭建 SkyWalking 服务。首先我们需要获取到SkyWalking的下载地址，官方下载地址如下：

[http://skywalking.apache.org/downloads/](https://links.jianshu.com/go?to=http%3A%2F%2Fskywalking.apache.org%2Fdownloads%2F)

这里我选择当前最新的6.6.0版本的二进制包：

![](/static/image/19037705-3ca72e9a4a2d7408.webp)

复制下载地址到服务器上进行下载并解压，具体步骤如下：

```
[root@localhost ~]# cd /usr/local/src
[root@localhost /usr/local/src]# wget http://mirrors.tuna.tsinghua.edu.cn/apache/skywalking/6.6.0/apache-skywalking-apm-6.6.0.tar.gz
[root@localhost /usr/local/src]# mkdir ../skywalking && tar -zxvf apache-skywalking-apm-6.6.0.tar.gz -C ../skywalking --strip-components 1
[root@localhost /usr/local/src]# cd ../skywalking/
[root@localhost /usr/local/skywalking]# ll -rh  # 解压后的目录文件如下
total 88K
drwxr-xr-x 2 root root   53 Dec 28 18:22 webapp
-rw-rw-r-- 1 1001 1002 2.0K Dec 24 14:10 README.txt
drwxrwxr-x 2 1001 1002  12K Dec 24 14:28 oap-libs
-rwxrwxr-x 1 1001 1002  32K Dec 24 14:10 NOTICE
drwxrwxr-x 3 1001 1002 4.0K Dec 28 18:22 licenses
-rwxrwxr-x 1 1001 1002  29K Dec 24 14:10 LICENSE
drwxr-xr-x 2 root root  221 Dec 28 18:22 config
drwxr-xr-x 2 root root  241 Dec 28 18:22 bin
drwxrwxr-x 8 1001 1002  143 Dec 24 14:21 agent
[root@localhost /usr/local/skywalking]#
```

运行bin目录下的**startup.sh**脚本即可启动skywalking服务：

```
[root@localhost /usr/local/skywalking]# bin/startup.sh
SkyWalking OAP started successfully!
SkyWalking Web Application started successfully!
[root@localhost /usr/local/skywalking]#
```

SkyWalking控制台服务默认监听8080端口，若有防火墙需要开放该端口：

```
[root@localhost /usr/local/skywalking]# firewall-cmd --zone=public --add-port=8080/tcp --permanent
success
[root@localhost /usr/local/skywalking]# firewall-cmd --reload
success
[root@localhost /usr/local/skywalking]#
```

若希望允许远程传输，则还需要开放11800（gRPC）和12800（rest）端口，远程agent将通过该端口传输收集的数据：

```
[root@localhost /usr/local/skywalking]# firewall-cmd --zone=public --add-port=11800/tcp --permanent
success
[root@localhost /usr/local/skywalking]# firewall-cmd --zone=public --add-port=12800/tcp --permanent
success
[root@localhost /usr/local/skywalking]# firewall-cmd --reload
success
[root@localhost /usr/local/skywalking]#
```

正常启动成功后，使用浏览器访问主页如下：  
![](/static/image/19037705-35f85f77e4bbde93.webp)

## 1.2.Windows搭建SkyWalking服务

Windows下的搭建就更简单了，首先下载Windows平台下的包：  
![](/static/image/19037705-478c92b0826f8a73.webp)

解压后目录文件如下：

![](/static/image/19037705-c729f98480b9825f.webp)  
双击bin目录下的**startup.bat**文件就可以运行SkyWalking服务了：  
![](/static/image/19037705-21e39d62e0c2d6c6.webp)

这里之所以介绍Windows下的搭建，是因为当SkyWalking收集服务部署在远程服务器上时，本地要进行调试的话得用到agent目录下的jar包：  
![](/static/image/19037705-0b4b13e649e1a2fc.webp)

该agent文件夹，可以单独复制出放在项目系统所在服务器的任意目录下。agent文件夹下的skywalking-agent.jar即为监控代理程序，只需要在jvm的启动命令中加载该jar包，即可完成监控代理。

## 1.4.服务链路追踪

目前主要的一些 APM 工具有: Cat、Zipkin、Pinpoint、SkyWalking，这里主要介绍 SkyWalking ，它是一款优秀的国产 APM 工具，包括了分布式追踪、性能指标分析、应用和服务依赖分析等。

在本文中主要介绍如何使用SkyWalking来实现服务链路追踪，关于服务链路追踪的概念在下文中已进行过说明，这里就不再赘述了：

* [Spring Cloud Sleuth + Zipkin 实现服务追踪](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.51cto.com%2Fzero01%2F2173394)

目前有多种工具可以实现服务链路追踪，主流的工具对比可以参考如下文章：

* [https://www.jianshu.com/p/0fbbf99a236e](https://www.jianshu.com/p/0fbbf99a236e)

以上小节完成了SkyWalking平台服务的搭建，接下来进入项目整合环节，将SkyWalking提供的agent与我们的项目进行整合，以达到监控目的。这里事先创建了两个简单的Spring Cloud项目，分别是consumer和producer：

### 1.4.1.项目整合

以上小节完成了SkyWalking平台服务的搭建，接下来进入项目整合环节，将SkyWalking提供的agent与我们的项目进行整合，以达到监控目的。这里事先创建了两个简单的Spring Cloud项目，分别是consumer和producer：

![](/static/image/19037705-d486cac540fdc965.webp)

这两个项目中均包含基础的组件依赖：nacos-discovery、openfeign及web。因为SkyWalking是通过Java agent这种语言探针的方式进行数据的收集和上传，所以不需要像zipkin那样添加额外的依赖和配置。

consumer将调用producer提供的接口，以达到后续在SkyWalking上展示一个简单的调用链路效果。故在producer中编写一个接口，代码如下：

```
@Slf4j
@RestController
@RequestMapping("/producer")
public class ProducerController {

    @GetMapping
    public String producer() {
        log.info("received a request");
        return "this message from producer";
    }
}
```

而consumer也有一个接口，该接口内则是调用了producer的接口。代码如下：

```
@Slf4j
@RestController
@RequiredArgsConstructor
@RequestMapping("/consumer")
public class ConsumerController {

    private final ProducerClient producerClient;

    @GetMapping
    public String consumer() {
        log.info("consumer something");
        // 通过feign调用
        String result = producerClient.producer();
        return "consumer: " + result;
    }
}
```

**ProducerClient**代码如下：

```
@FeignClient("producer")
public interface ProducerClient {

    @GetMapping("/producer")
    String producer();
}
```

完成代码编写后，接下来我们需要为每个服务配置一个agent，首先创建两个与producer和consumer服务对应的目录：

![](/static/image/19037705-9afd99497109a03e.webp)

然后将skywalking里的agent目录下的所有文件拷贝出来，分别粘贴到这两个新建的目录中：

![](/static/image/19037705-36b4bb52c3cd4632.webp)

![](/static/image/19037705-4858fc17e2421cec.webp)

接着分别编辑这两个目录下的**config/agent.config**文件，该文件是agent的配置文件。修改其中的服务名称，以及skywalking平台后端服务的连接地址。producer配置示例如下：

```
# The service name in UI 服务名称
agent.service_name=${SW_AGENT_NAME:producer}

# Backend service addresses. 收集后端服务的地址
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:192.168.0.71:11800}
```

consumer里的配置文件也需要按照如上示例进行修改，这里之所以分别拷贝了两个agent是为了让不同的服务使用不同的配置文件。

如果不想为每个服务都单独拷贝一个agent目录，则可以通过添加JVM启动参数来覆写配置项，这两种方式视实际情况选择即可。如下示例：

```
-javaagent:E:\skywalking\apache-skywalking-apm-bin\agent\skywalking-agent.jar
-Dskywalking.agent.service_name=consumer
-Dskywalking.collector.backend_service=192.168.0.71:11800
```

配置好agent之后，在IDEA中添加Spring Boot引导类的JVM参数，指定**skywalking-agent.jar**的目录路径：

![](/static/image/19037705-52712b48fc732a77.webp)

完成以上步骤后，分别启动producer和comsumer服务，请求/consumer接口，因为skywalking是懒加载的，需要进行请求才会连接收集服务：  
![](/static/image/19037705-5432fdc4357a3877.webp)

通过浏览器访问 [http://serverIP:8090](http://serverIP:8090) 默认的用户名密码为：admin/admin，登录成功后，效果如下图出现如下界面即表示启动成功  
![](/static/image/15663120-c00c56c9763ffc5f.webp)

接着到SkyWalking的“追踪”页面上，就可以查看到调用链路信息了。如下图所示：  
![](/static/image/19037705-4b9aa9a8582898d4.webp)

点击链路上的节点可以查看到对应的详情：  
![](/static/image/19037705-8e265849fa12b513.webp)


## 2.参考
SkyWalking 分布式追踪系统：https://www.jianshu.com/p/2fd56627a3cf


## 基于Nepxion Discovery灰度发布的步骤流程

采用的技术：

服务发现注册中心:spring cloud eureka,

配置中心：spring cloud zookeeper、apollo

第三方开源组件: Nepxion Discovery

#### 添加依赖

```
<!--添加依赖版本管理  -->
<properties>
<discovery.plugin.version>4.8.0-RC2</discovery.plugin.version>
<spring.cloud.alibaba.version>0.2.0.RELEASE</spring.cloud.alibaba.version>
</properties>
<dependencyManagement>
<dependencies>

<!--用做灰度发布  -->
<dependency>
<groupId>com.nepxion</groupId>
<artifactId>discovery</artifactId>
<version>${discovery.plugin.version}</version>
<type>pom</type>
<scope>import</scope>
</dependency>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-alibaba-dependencies</artifactId>
<version>${spring.cloud.alibaba.version}</version>
<type>pom</type>
<scope>import</scope>
</dependency>
</dependencies>
</dependencyManagement>



<!--灰度发布依赖-->
<dependency>
<groupId>com.nepxion</groupId>
<artifactId>discovery-plugin-strategy-starter-service</artifactId>
</dependency>
<dependency>
<groupId>com.nepxion</groupId>
<artifactId>discovery-plugin-starter-eureka</artifactId>
</dependency>
<dependency>
<groupId>com.nepxion</groupId>
<artifactId>discovery-plugin-config-center-starter-apollo</artifactId>
</dependency>
```

#### 添加配置\(可以启动命令配置或配置文件配置,请注意各语法\)

```
1.由于现有的服务各个节点都加上跨域的配置，所以需要在网关\(zuul\)里面配置

zuul.routes.路由名称.sensitiveHeaders: Access-Control-Allow-Origin,Access-Control-Allow-Methods

2.服务发现注册中心

eureka.instance.metadata-map.group=flight 名称自定义\(各个节点中的都需要保持在一个组内\)

eureka.instance.metadata-map.version=1.0  版本自定义\(用于各个服务版本的对应\)

3.apollo配置

apollo.meta=http://ip:8080

app.id=discovery
```

#### 使用Nepxion   Console

如果要使用Desktop Console ,则需要新建项目Console，有一点需要要注意，不要在项目中添加context-path ,如果加了context-path,在console中刷新灰度配置的信息的时候，会出现url错误

#### 关于apollo配置中心的安装及使用请参与以下链接

[https://github.com/ctripcorp/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E4%BB%8B%E7%BB%8D](https://github.com/ctripcorp/apollo/wiki/Apollo配置中心介绍)

1. apollo配置中心的使用

```
1.登录[http://ip:8070/signin](http://ip:8070/signin)

2.用户名:apollo密码: admin

3.添加项目 名称自定义

4.可以添加namespace,也可以使用默认的application

5.添加配置key的名称定义,group-serviceId，这些都是在各个服务配置文件或启动参数中配置,如flight-zuul,其中中flight是eureka.instance.metadata.group , serviceId就是spring.application.name

6.配置key对应的value请参与以下xml格式
<?xml version="1.0" encoding="UTF-8"?>
<rule>
<discovery>
<version>
<!--表示网关z的1.0，允许访问提供端服务a的1.0版本-->
<service consumer-service-name="flight-web-api" provider-service-name="scm" consumer-version-value="1.0" provider-version-value="1.1"/>
<service consumer-service-name="flight-web-api" provider-service-name="onlinecommon" consumer-version-value="1.0" provider-version-value="1.0"/>
</version>
</discovery>
</rule>
7.编写好配置之后，点击发布，相应的服务应用会接收到apollo配置变动，从而将灰度发布生效
```




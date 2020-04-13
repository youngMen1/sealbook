# 1.基本介绍

ureka通过运行多个实例，使其更具有高可用性。事实上，这是它默认的熟性，你需要做的就是给对等的实例一个合法的关联serviceurl。

## 2.怎么使用

在eureka-server工程中resources文件夹下，创建配置文件application-peer1.yml:

```
server:
  port: 8761

spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2:8769/eureka/
```

并且创建另外一个配置文件application-peer2.yml：

```
server:
  port: 8769

spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/
```

**按照官方文档的指示，需要改变etc/hosts，linux系统通过vim /etc/hosts ,加上：**

```
127.0.0.1 peer1
127.0.0.1 peer2
```

windows电脑，在c:/windows/systems/drivers/etc/hosts 修改。

这时需要改造下客服端service-hi:

```
eureka:
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/
server:
  port: 8762
spring:
  application:
    name: service-hi
```

## 启动工程 {#三启动工程}

**启动eureka-server：**

```
java -jar eureka-server-0.0.1-SNAPSHOT.jar - -spring.profiles.active=peer1

java -jar eureka-server-0.0.1-SNAPSHOT.jar - -spring.profiles.active=peer2
```

**启动service-hi:**

```
java -jar service-hi-0.0.1-SNAPSHOT.jar
```

**访问：localhost:8761,如图：**  
![img](/static/image/2279594-659c68e405bd70bd.png)

你会发现注册了service-hi，并且有个peer2节点，同理访问localhost:8769你会发现有个peer1节点。

client只向8761注册，但是你打开8769，你也会发现，8769也有 client的注册信息。

**个人感受：这是通过看官方文档的写的demo ，但是需要手动改host是不是不符合Spring Cloud 的高上大？**

**eureka.instance.preferIpAddress=true是通过设置ip让eureka让其他服务注册它。也许能通过去改变去通过改变host的方式。**

此时的架构图：  
![img](/static/image/2279594-a052854a3084fdd6.png)  
Eureka-eserver peer1 8761,Eureka-eserver peer2 8769相互感应，当有服务注册时，两个Eureka-eserver是对等的，它们都存有相同的信息，这就是通过服务器的冗余来增加可靠性，当有一台服务器宕机了，服务并不会终止，因为另一台服务存有相同的数据。

# 3.总结

**eureka.instance.preferIpAddress=true是通过设置ip让eureka让其他服务注册它。也许能通过去改变去通过改变host的方式：**

**application-dev1.yml:**

```
EUREKA_HOST: 172.18.xx.243
EUREKA_PORT: 22002
SERVER_PORT: 22001
spring:
  application:
    name: eureka-server-hotel
eureka:
  environment: dev
  server:
    enable-self-preservation: false
  instance:
      hostname: localhost
      status-page-url: http://${spring.cloud.client.ip-address}:${server.port}/swagger-ui.html
      prefer-ip-address: true
      instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
       defaultZone: http://${EUREKA_HOST:localhost}:${EUREKA_PORT:22002}/eureka/
server:
  port: ${SERVER_PORT:22001}

logging:
  file: /var/log/hotel-eureka/inf-eureka.log
```

**application-dev2.yml:**

```
EUREKA_HOST: 172.18.xx.226
EUREKA_PORT: 22001
SERVER_PORT: 22002
spring:
  application:
    name: eureka-server-hotel
eureka:
  environment: dev
  server:
    enable-self-preservation: false
  instance:
      hostname: localhost
      status-page-url: http://${spring.cloud.client.ip-address}:${server.port}/swagger-ui.html
      prefer-ip-address: true
      instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
       defaultZone: http://${EUREKA_HOST:localhost}:${EUREKA_PORT:22001}/eureka/
server:
  port: ${SERVER_PORT:22002}

logging:
  file: /var/log/hotel-eureka/inf-eureka.log
```

# 4.来源

[https://www.fangzhipeng.com/springcloud/2018/08/10/sc-f10-eureka.html](https://www.fangzhipeng.com/springcloud/2018/08/10/sc-f10-eureka.html)

源码：[https://github.com/youngMen1/springcloud-code/tree/master/springcloud-eureka](https://github.com/youngMen1/springcloud-code/tree/master/springcloud-eureka)


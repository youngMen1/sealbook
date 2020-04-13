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
# 3.总结

# 4.来源




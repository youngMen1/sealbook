# Introduction

gitbook中文文档

[https://chrisniael.gitbooks.io/gitbook-documentation/content/github/index.html](https://chrisniael.gitbooks.io/gitbook-documentation/content/github/index.html)

# 一、服务器安装服务 {#一-服务器安装服务}

## 211.159.189.XX: {#211159189XX}

### 服务: {#服务}

eureka:

nohup java -jar inf-eureka-1.0.0.jar --spring.profiles.active=native1

[http://211.159.189.XX:22001/](http://211.159.189.77:22001/)

mysql

xxl-job:[http://211.159.189.XX:8080/xxl-job-admin/](http://211.159.189.77:8080/xxl-job-admin/)

安装了docker

---

## 134.175.12.XXX: {#13417512XXX}

### 服务: {#服务-2}

eureka:

nohup java -jar inf-eureka-1.0.0.jar --spring.profiles.active=native2

[http://134.175.12.XXX:22002/](http://134.175.12.243:22002/)

#### rabbitMq: {#rabbitmq}

1 切换到root用户 sudo su root

2 执行重新启动的命令 rabbitmq-server restart

#### Spring Cloud Alibaba:

Nacos作为服务注册发现组件：

---

## 47.107.152.XX: {#4710715XXX}

nohup java -jar inf-eureka-1.0.0.jar --spring.profiles.active=native1

nohup java -jar /home/java/springcloud-eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=native1 &gt;&gt; /home/java/output.log 2&gt;&1 &

### 服务: {#服务-3}

nginx 封师傅官网:taocicom

![](/assets/微信截图_20190712114223.png)

nexus-3.15.2-01:

安装路径:/usr/local/nexus-3.15.2-01

zookeeper:

安装路径:/usr/local/zookeeper/zookeeper-3.4.14

---

## 120.79.XXX.221

eureka:nohup java -jar /home/java/springcloud-eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=native1 &gt;&gt; /home/java/output.log 2&gt;&1 &

---

![](/assets/微信图片_20190625154318.jpg)


# Introduction

gitbook中文文档

[https://chrisniael.gitbooks.io/gitbook-documentation/content/github/index.html](https://chrisniael.gitbooks.io/gitbook-documentation/content/github/index.html)

# 一、服务器安装服务 {#一-服务器安装服务}

## 211.159.189.77: {#21115918977}

### 服务: {#服务}

eureka:

nohup java -jar inf-eureka-1.0.0.jar --spring.profiles.active=native1

[http://211.159.189.77:22001/](http://211.159.189.77:22001/)

mysql

xxl-job:[http://211.159.189.77:8080/xxl-job-admin/](http://211.159.189.77:8080/xxl-job-admin/)

---

## 134.175.12.243: {#13417512243}

### 服务: {#服务-2}

eureka:

nohup java -jar inf-eureka-1.0.0.jar --spring.profiles.active=native2

[http://134.175.12.243:22002/](http://134.175.12.243:22002/)

#### rabbitMq: {#rabbitmq}

1 切换到root用户 sudo su root

2 执行重新启动的命令 rabbitmq-server restart

#### Spring Cloud Alibaba:

Nacos作为服务注册发现组件：[http://localhost:8848/nacos，会跳转到登陆界面，默认的登陆用户名为nacos，密码也为nacos](http://localhost:8848/nacos%EF%BC%8C%E4%BC%9A%E8%B7%B3%E8%BD%AC%E5%88%B0%E7%99%BB%E9%99%86%E7%95%8C%E9%9D%A2%EF%BC%8C%E9%BB%98%E8%AE%A4%E7%9A%84%E7%99%BB%E9%99%86%E7%94%A8%E6%88%B7%E5%90%8D%E4%B8%BAnacos%EF%BC%8C%E5%AF%86%E7%A0%81%E4%B9%9F%E4%B8%BAnacos%E3%80%82)

---

## 47.107.152.93: {#4710715293}

### 服务: {#服务-3}

nginx 封师傅官网:taocicom

![](/assets/微信截图_20190712114223.png)

nexus-3.15.2-01:

安装路径:/usr/local/nexus-3.15.2-01

zookeeper:

安装路径:/usr/local/zookeeper/zookeeper-3.4.14

---

![](/assets/微信图片_20190625154318.jpg)


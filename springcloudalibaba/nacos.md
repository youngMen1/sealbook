# 1.Nacos是什么

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。 是Spring Cloud A 中的服务注册发现组件，类似于Consul、Eureka，同时它又提供了分布式配置中心的功能，这点和Consul的config类似，支持热加载。  
Nacos 的关键特性包括:

* 服务发现和服务健康监测
* 动态配置服务，带管理界面，支持丰富的配置维度。
* 动态 DNS 服务
* 服务及其元数据管理

# 2.使用Nacos服务注册和发现

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
# 3.总结

Nacos下载地址:[https://github.com/alibaba/nacos/releases](https://github.com/alibaba/nacos/releases)

# 4.参考




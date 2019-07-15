# XXL-JOB分布式任务调度平台

## 一、概述

XXL-JOB是一个轻量级分布式任务调度框架，开箱即用。

# 二、特性 {#特性}

简单：通过Web页面操作简单易用。

任务实时监控，可查看任务执行日志。

动态：可以动态修改任务状态，暂停或恢复任务，也可以终止进行中的任务。

路由策略：第一个，最后一个，轮询，随机，分片广播，故障转移等。

邮件报警：任务失败时支持邮件报警，可配置多邮件地址群发报警。

运行报表：实时查看任务数量、调度次数、执行器数量。

# 三、准备

注意打成war包的需要准备这些:

Windows:

官网下载tomcat：

![](/assets/微信截图_20190712173009.png)

配置tomcat环境变量:

[https://jingyan.baidu.com/article/a3761b2bf2ee681577f9aa42.html](https://jingyan.baidu.com/article/a3761b2bf2ee681577f9aa42.html)

## 四、搭建项目

GitHub:[https://github.com/xuxueli/xxl-job/](https://github.com/xuxueli/xxl-job/)

文档地址:[http://www.xuxueli.com/xxl-job/\#/?id=%E3%80%8A%E5%88%86%E5%B8%83%E5%BC%8F%E4%BB%BB%E5%8A%A1%E8%B0%83%E5%BA%A6%E5%B9%B3%E5%8F%B0xxl-job%E3%80%8B](http://www.xuxueli.com/xxl-job/#/?id=《分布式任务调度平台xxl-job》)

![](/assets/微信截图_20190712172637.png)

maven命令打包成war包 或者jar包：

```
# 编译
mvn clean compile
# 打包 
mvn clean package
```

![](/assets/微信截图_20190712174255.png)

```
java -jar xxl-job-admin-2.1.1-SNAPSHOT.jar   或者  用idea工具打开启动xxl-job-admin项目
在浏览器输入:http://localhost:8080/xxl-job-admin/
```

![](/assets/微信截图_20190712182859.png)

添加执行器管理![](/assets/微信截图_20190715093751.png)

添加任务管理![](/assets/微信截图_20190715093810.png)

项目里面的配置

![](/assets/微信截图_20190715095844.png)

注意版本:[https://blog.csdn.net/u011706563/article/details/89258029](https://blog.csdn.net/u011706563/article/details/89258029)

因为我是xxl-job2.0.0以上版本配置的版本

```
         2.0.0以上版本配置的版本
        /**
         * #xxl-job 2.0.0以上版本配置
         */
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppName(appName);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);
```




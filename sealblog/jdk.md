# centos7安装jdk

# [rpm安装JDK方法](https://www.cnblogs.com/peizhe123/p/7520431.html)

由于版权原因，Linux发行版并没有包含官方版的Oracle JDK，必须自己从官网上下载安装。Oracle官网用Cookie限制下载方式，使得眼下只能用浏览器进行下载，使用其他方式可能会导致下载失败。但还是有方法可以在Linux进行下载的，本文以wget为例。

* 我们需要三个参数：**–no-check-certificate、–no-cookies、–header**，通过`man wget`命令可以查到。
* **用于禁止检查证书**

## 首先我们要找到要下载JDK的URL地址，例如：

1. [http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm](http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm)
   。这个地址可以去Orcale的官网找到。
2. 通过wget命令下载：




# centos7安装jdk

# [rpm安装JDK方法](https://www.cnblogs.com/peizhe123/p/7520431.html)

由于版权原因，Linux发行版并没有包含官方版的Oracle JDK，必须自己从官网上下载安装。Oracle官网用Cookie限制下载方式，使得眼下只能用浏览器进行下载，使用其他方式可能会导致下载失败。但还是有方法可以在Linux进行下载的，本文以wget为例。

* 我们需要三个参数：
  **–no-check-certificate、–no-cookies、–header**
  ，通过
  `man wget`
  命令可以查到。
* **用于禁止检查证书**




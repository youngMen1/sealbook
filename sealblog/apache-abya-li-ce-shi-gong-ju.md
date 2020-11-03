# 1.Apache ab压力测试工具

## 1.1.ab的原理

## **ab**是apachebench命令的缩写。

**ab的原理：**ab命令会创建多个并发访问线程，模拟多个访问者同时对某一URL地址进行访问。它的测试目标是基于URL的，因此，它既可以用来测试apache的负载压力，也可以测试nginx、lighthttp、tomcat、IIS等其它Web服务器的压力。

ab命令对发出负载的计算机要求很低，它既不会占用很高CPU，也不会占用很多内存。但却会给目标服务器造成巨大的负载，其原理类似CC攻击。自己测试使用也需要注意，否则一次上太多的负载。可能造成目标服务器资源耗完，严重时甚至导致死机。

# 2.参考

Apache自带的ab压力测试工具用法详解：  
[https://blog.csdn.net/u011277123/article/details/78913649](https://blog.csdn.net/u011277123/article/details/78913649)


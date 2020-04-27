# 1.基本介绍

# 1.1.线程池简介

（1）、降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁造成的消耗；

  


 （2）、提高系统响应速度，当有任务到达时，通过复用已存在的线程，无需等待新线程的创建便能立即执行；

  


 （3）方便线程并发数的管控。因为线程若是无限制的创建，可能会导致内存占用过多而产生OOM，并且会造成cpu过度切换（cpu切换线程是有时间成本的（需要保持当前执行线程的现场，并恢复要执行线程的现场））。

  


 （4）提供更强大的功能，延时定时线程池。

  


6024478-88ee7b20f8f45825.webp

281039482656686.png

# 2.怎么使用

# 3.参考

java 线程池 使用实例：  
[https://www.cnblogs.com/GarfieldEr007/p/10230865.html](https://www.cnblogs.com/GarfieldEr007/p/10230865.html)

Java线程池详解：

[https://www.jianshu.com/p/7726c70cdc40](https://www.jianshu.com/p/7726c70cdc40)

ThreadPoolExcutor线程池使用总结：

[https://blog.csdn.net/weixin\_39352976/article/details/100884832](https://blog.csdn.net/weixin_39352976/article/details/100884832)


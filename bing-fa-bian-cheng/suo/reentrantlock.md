# 1.ReentrantLock源码分析

ReentrantLock重入锁，是实现Lock接口的一个类，也是在实际编程中使用频率很高的一个锁，**支持重入性，表示能够对共享资源能够重复加锁，即当前线程获取该锁再次获取不会被阻塞**。在java关键字synchronized隐式支持重入性（关于synchronized可以[看这篇文章](https://juejin.im/post/5ae6dc04f265da0ba351d3ff)），synchronized通过获取自增，释放自减的方式实现重入。与此同时，ReentrantLock还支持**公平锁和非公平锁**两种方式。那么，要想完完全全的弄懂ReentrantLock的话，主要也就是ReentrantLock同步语义的学习：1. 重入性的实现原理；2. 公平锁和非公平锁。


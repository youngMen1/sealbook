# 1.基本介绍
FindBug 是一款开源的 Java 代码检查工具,遵循 GNU 公共许可协议。它可以检查 Java 类或者 JAR 文件,运行的是 Java 字节码而不是源码,检查原理是:将字节码与一组缺陷模式进行对比来发现可能存在的问题,这些问题包括空指针引用、无限递归循环、死锁等。检查的 bug 类型包括:
![img](/static/image/2139461-2b1f7a4e8aa911a7.webp)
* Malicious code vulnerability：恶意代码
* Dodgy code：不符合规范的代码
* Internationalization：国际化相关问题，如错误的字符串转换;
* Bad practice：坏的实践:常见代码错误,序列化错误,用于静态代码检查时进行缺陷模式匹配;
* Multithreaded correctness：多线程的正确性:如多线程编程时常见的同步,线程调度问题;
* Performance：运行时性能问题，如由变量定义,方法调用导致的代码低效问题。
* Correctness：可能导致错误的代码,如空指针引用等;
* Experimental：可能受到的恶意攻击,如访问权限修饰符的定义等;
* Security：安全性


1.Of Concren 建议, 如果遵循能更好的完善代码

2.Troubling 不好的, 可能会引发不良后果

3.Scary 严重问题, 在某种情况下一定会出现问题

4.Scariest 非常严重, 已经影响到当前程序功能

可以按照严重级别倒序进行修复, 如果时间允许, 可以将 Of Concren 中的问题也一并修复

![img](/static/image/20180720151945178.png)




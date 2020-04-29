# 1.基本介绍

FindBug 是一款开源的 Java 代码检查工具,遵循 GNU 公共许可协议。它可以检查 Java 类或者 JAR 文件,运行的是 Java 字节码而不是源码,检查原理是:将字节码与一组缺陷模式进行对比来发现可能存在的问题,这些问题包括空指针引用、无限递归循环、死锁等。检查的 bug 类型包括:  
![img](/static/image/2139461-2b1f7a4e8aa911a7.webp)

* Malicious code vulnerability：恶意代码
* Dodgy code：不符合规范的代码
* Internationalization：国际化相关问题，如错误的字符串转换;
* Bad practice：坏的实践：常见代码错误,序列化错误,用于静态代码检查时进行缺陷模式匹配;
* Multithreaded correctness：多线程的正确性:如多线程编程时常见的同步,线程调度问题;
* Performance：运行时性能问题，如由变量定义,方法调用导致的代码低效问题。
* Correctness：可能导致错误的代码,如空指针引用等;
* Experimental：可能受到的恶意攻击,如访问权限修饰符的定义等;
* Security：安全性

```
1. Bad practice 坏的实践

  一些不好的实践，下面列举几个： 
1）类定义了equals()，却没有hashCode()；。 
2）Statement 的execute方法调用了非常量的字符串；或Prepared Statement是由一个非常量的字符串产生。 
3）方法终止或不处理异常，一般情况下，异常应该被处理或报告，或被方法抛出。

2. Correctness 一般的正确性问题

  可能导致错误的代码，下面列举几个： 
1）空指针被引用；在方法的异常路径里，空指针被引用；方法没有检查参数是否null；null值产生并被引用；null值产生并在方法的异常路径被引用；传给方法一个声明为@NonNull的null参数；方法的返回值声明为@NonNull实际是null。 
2）类定义了hashcode()方法，但实际上并未覆盖父类Object的hashCode()；类定义了tostring()方法，但实际上并未覆盖父类Object的toString()；很明显的方法和构造器混淆；方法名容易混淆。 
3）方法尝试访问一个Prepared Statement的0索引；方法尝试访问一个ResultSet的0索引。 
4）所有的write都把属性置成null，这样所有的读取都是null，这样这个属性是否有必要存在；或属性从没有被write。

3. Internationalization 国际化

  当对字符串使用upper或lowercase方法，如果是国际的字符串，可能会不恰当的转换。

4. Malicious code vulnerability 恶意代码 

  如果代码公开，可能受到恶意攻击的代码，下面列举几个： 
1）一个类的finalize()应该是protected，而不是public的。 
2）属性是可变的数组；属性是可变的Hashtable；属性应该是package protected的。

5. Multithreaded correctness 多线程的正确性

  多线程编程时，可能导致错误的代码，下面列举几个： 
1）ESync：空的同步块，很难被正确使用。 
2）MWN：错误使用notify()，可能导致IllegalMonitorStateException异常；或错误的 
使用wait()。 
3）使用notify()而不是notifyAll()，只是唤醒一个线程而不是所有等待的线程。 
4）构造器调用了Thread.start()，当该类被继承可能会导致错误。

6. Performance 性能问题

  可能导致性能不佳的代码，下面列举几个： 
1）DM：方法调用了低效的Boolean的构造器，而应该用Boolean.valueOf(…)；用类似 
Integer.toString(1) 代替new Integer(1).toString()；方法调用了低效的float的构造器，应该用静态的valueOf方法。 
2）SIC：如果一个内部类想在更广泛的地方被引用，它应该声明为static。 
3）SS： 如果一个实例属性不被读取，考虑声明为static。 
4）UrF：如果一个属性从没有被read，考虑从类中去掉。 
5）UuF：如果一个属性从没有被使用，考虑从类中去掉。

7. Dodgy 不符合规范的，有潜在危险的

  具有潜在危险的代码，可能运行期产生错误，下面列举几个： 
1）CI： 类声明为final但声明了protected的属性。 
2）DLS：对一个本地变量赋值，但却没有读取该本地变量；本地变量赋值成null，却没有读取该本地变量。 
3）ICAST： 整型数字相乘结果转化为长整型数字，应该将整型先转化为长整型数字再相乘。 
4）INT：没必要的整型数字比较，如X <= Integer.MAX_VALUE。 
5）NP： 对readline()的直接引用，而没有判断是否null；对方法调用的直接引用，而方法可能返回null。 
6）REC：直接捕获Exception，而实际上可能是RuntimeException。 
7）ST： 从实例方法里直接修改类变量，即static属性。
```

## 1.2.BUG 严重级别

![img](/static/image/20180720151945178.png)  
1.Of Concren 建议, 如果遵循能更好的完善代码

2.Troubling 不好的, 可能会引发不良后果

3.Scary 严重问题, 在某种情况下一定会出现问题

4.Scariest 非常严重, 已经影响到当前程序功能

可以按照严重级别倒序进行修复, 如果时间允许, 可以将 Of Concren 中的问题也一并修复

## 1.3.图上标注的含义

### 1.3.1.检查范围

标注1：检查当前类（只有在选中类的时候可点击）。  
标注2：检查当前包。  
标注3：检查当前模块。  
标注4：检查当前项目。  
标注5：自定义设置检查范围.

### 1.3.2.结果查看方式

标注6：按照bug类型查看。  
标注7：按照类查看。  
标注8：按照包结构查看。  
标注9：按照bug等级查看。

![img](/static/image/2139461-b1da78ef6089908d.webp)


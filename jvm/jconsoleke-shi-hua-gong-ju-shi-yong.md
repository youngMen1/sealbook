# 1.基本介绍

Jconsole （Java Monitoring and Management Console），一种基于JMX的可视化监视、管理工具。

## 1.1.启动JConsole {#12-启动jconsole}

点击JDK/bin 目录下面的

* `jconsole.exe`
  即可启动
* 然后会自动自动搜索本机运行的所有虚拟机进程。
* 选择其中一个进程可开始进行监控

![img](/static/image/20180422221307379)

# 2.怎么使用

## 2.1.JConsole功能基本介绍

JConsole 基本包括以下基本功能：概述、内存、线程、类、VM概要、MBean

运行下面的程序、然后使用JConsole进行监控;注意设置虚拟机参数

```
import java.util.ArrayList;
import java.util.List;

/**
 *  设置虚拟机参数：
 *  -Xms100M -Xms100m -XX:+UseSerialGC -XX:+PrintGCDetails
 */
public class JConsoleTool {

    static class OOMObject {
        public byte[] placeholder = new byte[64 * 1024];
    }

    public static void fillHeap(int num) throws InterruptedException {
        Thread.sleep(20000); //先运行程序，在执行监控
        List<OOMObject> list = new ArrayList<OOMObject>();
        for (int i = 0; i < num; i++) {
            // 稍作延时，令监视曲线的变化更加明显
            Thread.sleep(50);
            list.add(new OOMObject());
        }
        System.gc();
    }

    public static void main(String[] args) throws Exception {
        fillHeap(1000);
        while(true){
            //让其一直运行着
        }

    }
}
```

## 1.3.内存监控

内存页签相对于可视化的jstat 命令，用于监视受收集器管理的虚拟机内存。  
![img](/static/image/20180423161143839)

选项    描述

Eden Space 的大小    27328KB

已用    正在使用

已提交    27328KB

最大值    27328KB

copy 上的 0.120s\(3收集\)    新生代使用赋值算法（copy）,0.120s,总共三次

MarkSweepCompact上的 0.037（1收集）    老年代使用标记清除整理，耗时0.037，总共一次

对应的GC日志：

```
[GC (Allocation Failure) [DefNew: 27277K->3392K(30720K), 0.0349173 secs] 27277K->14749K(99008K), 0.0350411 secs] [Times: user=0.03 sys=0.00, real=0.04 secs] 

[GC (Allocation Failure) [DefNew: 30691K->3378K(30720K), 0.0446635 secs] 42049K->39217K(99008K), 0.0447387 secs] [Times: user=0.03 sys=0.01, real=0.04 secs] 

[GC (Allocation Failure) [DefNew: 30679K->3372K(30720K), 0.0408609 secs] 66518K->64734K(99008K), 0.0409604 secs] [Times: user=0.02 sys=0.02, real=0.04 secs] 

[Full GC (System.gc()) [Tenured: 61362K->66352K(68288K), 0.0372192 secs] 67024K->66352K(99008K), [Metaspace: 9535K->9535K(1058816K)], 0.0373411 secs] [Times: user=0.05 sys=0.00, real=0.04 secs]
```

## 1.4.线程监控

```
如果上面的“内存”页签相当于可视化的jstat命令的话，“线程”页签的功能相当于可视化的jstack命令，
遇到线程停顿时可以使用这个页签进行监控分析。
线程长时间停顿的主要原因主要有：等待外部资源（数据库连接、网络资源、设备资 
源等）、死循环、锁等待（活锁和死锁）
```

**下面三个方法分别等待控制台输入、死循环演示、线程锁等待演示：**

```
/**
  * 等待控制台输入
  * @throws IOException
  */
 public static  void waitRerouceConnection () throws IOException {
     BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
     br.readLine();
 }
/**
 * 线程死循环演示
 */
 public static void createBusyThread() {
     Thread thread = new Thread(new Runnable() {
         @Override
         public void run() {
             while (true)   // 第41行
                 ;
         }
     }, "testBusyThread");
     thread.start();
 }

 /**
  * 线程锁等待演示
  */
 public static void createLockThread(final Object lock) {
     Thread thread = new Thread(new Runnable() {
         @Override
         public void run() {
             synchronized (lock) {
                 try {
                     lock.wait();
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
             }
         }
     }, "testLockThread");
     thread.start();
 }
```

**线程死锁演示：**  
这段代码开了200个线程去分别计算1+2以及2+1的值，其实for循环是可省略的，两个线程也可能会导致死锁，不过那样概率太小，需要尝试运行很多次才能看到效果。一般的话，带for循环的版本最多运行2～3次就会遇到线程死锁，程序无法结束。造成死锁的原因是Integer.valueOf（）方法基于减少对象创建次数和节省内存的考虑，\[-128，127\]之间的数字会被缓存\[3\]，当valueOf（）方法传入参数在这个范围之内，将直接返回缓存中的对象。也就是说，代码中调用了200次Integer.valueOf（）方法一共就只返回了两个不同的对象。假如在某个线程的两个synchronized块之间发生了一次线程切换，那就会出现线程A等着被线程B持有的Integer.valueOf（1），线程B又等着被线程A持有的Integer.valueOf（2），结果出现大家都  
跑不下去的情景。

```
package com.jvm;

/**
 * 线程死锁验证
 */
public class JConsoleThreadLock {

    /**
     * 线程死锁等待演示
     */
    static class SynAddRunalbe implements Runnable {
        int a, b;
        public SynAddRunalbe(int a, int b) {
            this.a = a;
            this.b = b;
        }

        @Override
        public void run() {
            synchronized (Integer.valueOf(a)) {
                synchronized (Integer.valueOf(b)) {
                    System.out.println(a + b);
                }
            }
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(new SynAddRunalbe(1, 2)).start();
            new Thread(new SynAddRunalbe(2, 1)).start();
        }
    }
}
```

![img](/static/image/20180423165405892)  
结果描述：显示了线程Thread-53在等待一个被线程Thread-66持有Integer对象，而点击线程Thread-66则显示它也在等待一个Integer对象，被线程Thread-53持有，这样两个线程就互相卡住，都不存在等到锁释放的希望了  
**VisualVM（All-in-One Java Troubleshooting Tool）;功能最强大的运行监视和故障处理程序**

# VisualVM介绍

## 2.1 功能描述

显示虚拟机进程以及进程的配置、环境信息（jps、jinfo）。  
监视应用程序的CPU、GC、堆、方法区\(1.7及以前\)，元空间（JDK1.8及以后）以及线程的信息（jstat、jstack）。  
dump以及分析堆转储快照（jmap、jhat）。  
方法级的程序运行性能分析，找出被调用最多、运行时间最长的方法。  
离线程序快照：收集程序的运行时配置、线程dump、内存dump等信息建立一个快照

## 2.2 使用教程

如何使用，直接查看官网和本书教程即可。

# 

# 2.总结

# 4.参考

VisualVM官网地址：帮助文档  
BTrace 简要介绍  
《深入理解java虚拟机》–周志明

JConsole可视化工具介绍：  
[https://blog.csdn.net/qq\_31156277/article/details/80035430](https://blog.csdn.net/qq_31156277/article/details/80035430)


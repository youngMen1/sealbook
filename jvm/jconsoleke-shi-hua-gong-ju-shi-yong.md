# 1.基本介绍

Jconsole （Java Monitoring and Management Console），一种基于JMX的可视化监视、管理工具。

## 1.1.启动JConsole {#12-启动jconsole}

点击JDK/bin 目录下面的

* `jconsole.exe`
  即可启动
* 然后会自动自动搜索本机运行的所有虚拟机进程。
* 选择其中一个进程可开始进行监控

![img](/static/image/20180422221307379)

## 1.2.JConsole功能基本介绍

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

如果上面的“内存”页签相当于可视化的jstat命令的话，“线程”页签的功能相当于可视化的jstack命令，遇到线程停顿时可以使用这个页签进行监控分析。线程长时间停顿的主要原因主要有：

`等待外部资源`

（数据库连接、网络资源、设备资

  


源等）、

`死循环`

、

`锁等待`

（活锁和死锁）

# 3.怎么使用

# 2.总结

# 4.参考

JConsole可视化工具介绍：  
[https://blog.csdn.net/qq\_31156277/article/details/80035430](https://blog.csdn.net/qq_31156277/article/details/80035430)


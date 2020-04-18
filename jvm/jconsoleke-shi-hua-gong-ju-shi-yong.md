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

# 1.3.内存监控

内存页签相对于可视化的jstat 命令，用于监视受收集器管理的虚拟机内存。  
![img](/static/image/20180423161143839)

选项	描述

Eden Space 的大小	27328KB

已用	正在使用

已提交	27328KB

最大值	27328KB

copy 上的 0.120s\(3收集\)	新生代使用赋值算法（copy）,0.120s,总共三次

MarkSweepCompact上的 0.037（1收集）	老年代使用标记清除整理，耗时0.037，总共一次



# 3.怎么使用

# 2.总结

# 4.参考

JConsole可视化工具介绍：  
[https://blog.csdn.net/qq\_31156277/article/details/80035430](https://blog.csdn.net/qq_31156277/article/details/80035430)


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



# 3.怎么使用

# 2.总结

# 4.参考

[https://blog.csdn.net/qq\_31156277/article/details/80035430](https://blog.csdn.net/qq_31156277/article/details/80035430)


# 1.JProfiler是什么 {#1}

JProfiler是由ej-technologies GmbH公司开发的一款性能瓶颈分析工具\(该公司还开发部署工具\)。  
其特点:

* 使用方便
* 界面操作友好
* 对被分析的应用影响小
* CPU,Thread,Memory分析功能尤其强大
* 支持对jdbc,noSql, jsp, servlet, socket等进行分析
* 支持多种模式\(离线，在线\)的分析
* ![img](/static/image/f71a75090d48e46eb809001918d37d7cc8d5ec90.png)
* 跨平台

## 二.数据采集 {#2}

Q1. JProfiler既然是一款性能瓶颈分析工具，这些分析的相关数据来自于哪里？  
Q2. JProfiler是怎么将这些数据收集并展现的?
![img](/static/image/774e1de366c3dced5bf97ab0cd34471ec9a99537.png)
(图2)

A1. 分析的数据主要来自于下面俩部分
1. 一部分来自于jvm的分析接口**JVMTI**(JVM Tool Interface) , JDK必须>=1.6


```
JVMTI is an event-based system. The profiling agent library can register handler functions for different events. It can then enable or disable selected events
```

例如: 对象的生命周期，thread的生命周期等信息
2. 一部分来自于instruments classes(可理解为class的重写,增加JProfiler相关统计功能)
例如：方法执行时间，次数，方法栈等信息
A2. 数据收集的原理如图2
1. 用户在JProfiler GUI中下达监控的指令(一般就是点击某个按钮)
2. JProfiler GUI JVM 通过socket(默认端口8849)，发送指令给被分析的jvm中的JProfile Agent。
3. JProfiler Agent(如果不清楚Agent请看文章第三部分"启动模式") 收到指令后，将该指令转换成相关需要监听的事件或者指令,来注册到JVMTI上或者直接让JVMTI去执行某功能(例如dump jvm内存)
4. JVMTI 根据注册的事件，来收集当前jvm的相关信息。 例如: 线程的生命周期; jvm的生命周期;classes的生命周期;对象实例的生命周期;堆内存的实时信息等等
5. JProfiler Agent将采集好的信息保存到**内存**中，按照一定规则统计好(如果发送所有数据JProfiler GUI，会对被分析的应用网络产生比较大的影响)
6. 返回给JProfiler GUI Socket.
7. JProfiler GUI Socket 将收到的信息返回 JProfiler GUI Render
8. JProfiler GUI Render 渲染成最终的展示效果
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

## 1.1.数据采集 {#2}

Q1. JProfiler既然是一款性能瓶颈分析工具，这些分析的相关数据来自于哪里？  
Q2. JProfiler是怎么将这些数据收集并展现的?  
![img](/static/image/774e1de366c3dced5bf97ab0cd34471ec9a99537.png)  
\(图2\)

**A1. 分析的数据主要来自于下面俩部分**  
1. 一部分来自于jvm的分析接口**JVMTI**\(JVM Tool Interface\) , JDK必须&gt;=1.6

```
JVMTI is an event-based system. The profiling agent library can register handler functions for different events. It can then enable or disable selected events
```

例如: 对象的生命周期，thread的生命周期等信息  
2. 一部分来自于instruments classes\(可理解为class的重写,增加JProfiler相关统计功能\)  
例如：方法执行时间，次数，方法栈等信息  
**A2. 数据收集的原理如图2**  
1. 用户在JProfiler GUI中下达监控的指令\(一般就是点击某个按钮\)  
2. JProfiler GUI JVM 通过socket\(默认端口8849\)，发送指令给被分析的jvm中的JProfile Agent。  
3. JProfiler Agent\(如果不清楚Agent请看文章第三部分"启动模式"\) 收到指令后，将该指令转换成相关需要监听的事件或者指令,来注册到JVMTI上或者直接让JVMTI去执行某功能\(例如dump jvm内存\)  
4. JVMTI 根据注册的事件，来收集当前jvm的相关信息。 例如: 线程的生命周期; jvm的生命周期;classes的生命周期;对象实例的生命周期;堆内存的实时信息等等  
5. JProfiler Agent将采集好的信息保存到**内存**中，按照一定规则统计好\(如果发送所有数据JProfiler GUI，会对被分析的应用网络产生比较大的影响\)  
6. 返回给JProfiler GUI Socket.  
7. JProfiler GUI Socket 将收到的信息返回 JProfiler GUI Render  
8. JProfiler GUI Render 渲染成最终的展示效果

## 1.2. 数据采集方式和启动模式

**A1. JProfier采集方式分为两种：**Sampling\(样本采集\)和Instrumentation

Sampling: 类似于样本统计, 每隔一定时间\(5ms\)将每个线程栈中方法栈中的信息统计出来。优点是对应用影响小\(即使你不配置任何Filter, Filter可参考文章第四部分\)，缺点是一些数据/特性不能提供\(例如:方法的调用次数\)

Instrumentation: 在class加载之前，JProfier把相关功能代码写入到需要分析的class中，对正在运行的jvm有一定影响。优点: 功能强大，但如果需要分析的class多，那么对应用影响较大，一般配合Filter一起使用。所以一般JRE class和framework的class是在Filter中通常会过滤掉。

注: JProfiler本身没有指出数据的采集类型，这里的采集类型是针对方法调用的采集类型 。因为JProfiler的绝大多数核心功能都依赖方法调用采集的数据, 所以可以直接认为是JProfiler的数据采集类型。

**A2: 启动模式:**

**Attach mode  **  
可直接将本机正在运行的jvm加载JProfiler Agent. 优点是很方便，缺点是一些特性不能支持。如果选择Instrumentation数据采集方式，那么需要花一些额外时间来重写需要分析的class。

**Profile at startup  **  
在被分析的jvm启动时，将指定的JProfiler Agent手动加载到该jvm。JProfiler GUI 将收集信息类型和策略等配置信息通过socket发送给JProfiler Agent，收到这些信息后该jvm才会启动。  
在被分析的jvm 的启动参数增加下面内容：  
语法: -agentpath:\[path to jprofilerti library\]  
【注】: 语法不清楚没关系，JProfiler提供了帮助向导.  
![img](/static/image/af3a9d42a43abf41a676e194dad2524c651b213c.png)  
\(图3\)

**Prepare for profiling:**  
和Profile at startup的主要区别：被分析的jvm不需要收到JProfiler GUI 的相关配置信息就可以启动。

**Offline profiling**  
一般用于适用于不能直接调试线上的场景。Offline profiling需要将信息采集内容和策略\(一些Trigger, Trigger请参考文章第五部分\)打包成一个配置文件\(config.xml\)，在线上启动该jvm 加载 JProfiler Agent时，加载该xml。那么JProfiler Agent会根据Trigger的类型会生成不同的信息。例如: heap dump; thread dump; method call record等  
语法:  
-agentpath:/home/2080/jprofiler8/bin/linux-x64/libjprofilerti.so=offline,id=151,config=/home/2080/config.xml  
【注】: config.xml中的每一个被分析的jvm的采集信息都有一个id来标识。  
下面是使用了离线模式，并使用了每隔一秒dump heap 的Trigger：  
![img](/static/image/93ca30653b599d9a8564dd05e3971d8078e9ec16.png)

## 1.3.JProfiler核心概念。

Filter: 什么class需要被分析。分为包含和不包含两种类型的Filter。  
![img](/static/image/c6490044d51af9e36d86c7c59774a26bf68934d8.png)  
Profiling Settings: 收据收集的策略:Sampling和 Instrumentation，一些数据采集细节可以自定义.  
![img](/static/image/267a5a432ded52a220742608122e125f73810db1.png)  
\(图5\)

Triggers: 一般用于**offline**模式，告知JProfiler Agent 什么时候触发什么行为来收集指定信息.  
![img](/static/image/e69504d0b635fae209f5672e0b2a271b5354e87a.png)  
\(图6\)

Live memory: class/class instance的相关信息。 例如对象的个数，大小，对象创建的方法执行栈，对象创建的热点。  
![img](/static/image/e8a52b590d0f4631058ca328fe0ff691c0d3aa89.png)  
\(图7\)

Heap walker: 对一定时间内收集的内存对像信息进行静态分析，功能强大且使用。包含对象的outgoing reference, incoming reference, biggest object等  
![img](/static/image/c85b6b5e6880bab6b7ead4b0a673b8e1575b0158.png)  
\(图8\)

CPU views: CPU消耗的分布及时间\(cpu时间或者运行时间\); 方法的执行图; 方法的执行统计\(最大，最小，平均运行时间等\)  
![img](/static/image/31d669359bf4291f61c8ba7436374134936994ba.png)  
\(图9\)

Thread: 当前jvm所有线程的运行状态，线程持有锁的状态，可dump线程。  
![img](/static/image/b8fef844181952665612a3ae9a23864d8eb0ec01.png)  
\(图10\)

Monitors & locks: 所有线程持有锁的情况以及锁的信息  
![img](/static/image/72e01bf3f2bdec2f6b05ce156a379ecb913f89e0.png)  
\(图11\)

Telemetries: 包含heap, thread, gc, class等的趋势图\(遥测视图\)

# 实践

为了方便实践，直接以JProfiler8自带的一个例子来帮助理解上面的相关概念。  
JProfiler 自带的例子如下：模拟了内存泄露和线程阻塞的场景：  
具体源码参考: /jprofiler install path/demo/bezier  
![img](/static/image/e95ff007af328eb31b4f9fb4d9d888bffdfe1d29.png)  
![img](/static/image/375e6515445717a1bb110738a17e61ee3de1e2aa.png)  
\(图13 Leak Memory 模拟内存泄露, Simulate blocking 模拟线程间锁的阻塞\)

A1. 首先来分析下内存泄露的场景:\(勾选图13中 Leak Memory 模拟内存泄露\)  
1. 在**Telemetries-&gt; Memory**视图中你会看到大致如下图的场景\(在看的过程中可以间隔一段时间去执行Run GC这个功能\)：看到下图蓝色区域,老生代在gc后\(**波谷**\)内存的大小在慢慢的增加\(理想情况下，这个值应该是稳定的\)  
![img](/static/image/c4c3c21a29874988408786f3c62f2953f713594a.png)  
\(图14\)

在 Live memory-&gt;Recorded Objects 中点击**record allocation data**按钮，开始统计一段时间内创建的对象信息。执行一次**Run GC**后看看当前对象信息的大小，并点击工具栏中**Mark Current**按钮\(其实就是给当前对象数量打个标记。执行一次Run GC，然后再继续观察;执行一次Run GC，然后再继续观察...。最后看看哪些对象在不断GC后，数量还一直上涨的。最后你看到的信息可能和下图类似  
65ae84a68f6d81a46064d55fb88eb658742f3241.png  
\(图15 绿色是标记前的数量，红色是标记后的增量\)

在Heap walker中分析刚才记录的对象信息  
![img](/static/image/de3a4d18921259d1767bd7ac0c05fe02678b3c26.png)  
\(图16\)  
![img](/static/image/a0693872e6fedb18d4ce3b94dea07bef2c35468e.png)  
\(图17\)

点击上图中实例最多的class，右键**Use Selected Instances-&gt;Reference-&gt;Incoming Reference**.  
发现该Long数据最终是存放在**bezier.BeaierAnim.leakMap**中。  
![img](/static/image/de60d0d5dcb88017ed0dbda668b921c43bca869a.png)  
\(图18\)

在Allocations tab项中，右键点击其中的某个方法，可查看到具体的源码信息.

![img](/static/image/ffb374dd45b1cf006f689fa56d2f4fbff3c85d1c.png)  
\(图19\)

【注】:到这里问题已经非常清楚了，明白了在图17中为什么哪些实例的数量是一样多，并且为什么内存在fullgc后还是回收不了\(一个old 区的对象leakMap，put的信息也会进入old区, leakMap如回收不掉，那么该map中包含的对象也回收不掉\)。

A2. 模拟线程阻塞的场景\(勾选图13中Simulate blocking 模拟线程间锁的阻塞\)  
为了方便区分线程，我将Demo中的BezierAnim.java的L236的线程命名为test

```
public void start() {
            thread = new Thread(this, "test");
            thread.setPriority(Thread.MIN_PRIORITY);
            thread.start();
        }
```
正常情况下，如下图
603df4fd16b151739dbafedc0631fd3036b5fe36.png
(图20)

勾选了Demo中"Simulate blocking"选项后，如下图(注意看下下图中的状态图标), test线程block状态明显增加了。

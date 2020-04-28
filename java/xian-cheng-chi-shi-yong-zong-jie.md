# 1.基本介绍

## 1.1.线程池简介

（1）、降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁造成的消耗；

（2）、提高系统响应速度，当有任务到达时，通过复用已存在的线程，无需等待新线程的创建便能立即执行；

（3）方便线程并发数的管控。因为线程若是无限制的创建，可能会导致内存占用过多而产生OOM，并且会造成cpu过度切     换（cpu切换线程是有时间成本的（需要保持当前执行线程的现场，并恢复要执行线程的现场））。

（4）提供更强大的功能，延时定时线程池。

**线程池流程：**

![img](/static/image/6024478-88ee7b20f8f45825.webp)

**线程池的类体系结构：**

![img](/static/image/281039482656686.png)

**标记一下比较重要的类：**

| ExecutorService | 真正的线程池接口。 |
| :--- | :--- |
| ScheduledExecutorService | 能和Timer/TimerTask类似，解决那些需要任务重复执行的问题。 |
| ThreadPoolExecutor | ExecutorService的默认实现。 |
| ScheduledThreadPoolExecutor | 继承ThreadPoolExecutor的ScheduledExecutorService接口实现，周期性任务调度的类实现。 |

## 1.2.为什么用线程池

1.创建/销毁线程伴随着系统开销，过于频繁的创建/销毁线程，会很大程度上影响处理效率

```
例如：
记创建线程消耗时间T1，执行任务消耗时间T2，销毁线程消耗时间T3
如果T1+T3>T2，那么是不是说开启一个线程来执行这个任务太不划算了！
正好，线程池缓存线程，可用已有的闲置线程来执行新任务，避免了T1+T3带来的系统开销
```

2.线程并发数量过多，抢占系统资源从而导致阻塞

```
我们知道线程能共享系统资源，如果同时执行的线程过多，就有可能导致系统资源不足而产生阻塞的情况运用线程池能有效的控制线程最大并发数，避免以上的问题
```

3.对线程进行一些简单的管理

```
比如：延时执行、定时循环执行的策略等

运用线程池都能进行很好的实现
```

## 1.2.线程池ThreadPoolExecutor

### 1.2.1.ThreadPoolExecutor提供了四个构造函数

```
// 五个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue)

// 六个参数的构造函数-1
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory)

// 六个参数的构造函数-2
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          RejectedExecutionHandler handler)

// 七个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

* **int corePoolSize      **该线程池中**核心线程数最大值**

```
核心线程：

线程池新建线程的时候，如果当前线程总数小于corePoolSize，则新建的是核心线程，如果超过corePoolSize，则新建的是非核心线程

核心线程默认情况下会一直存活在线程池中，即使这个核心线程啥也不干(闲置状态)。

如果指定ThreadPoolExecutor的allowCoreThreadTimeOut这个属性为true，那么核心线程如果不干活(闲置状态)的话，超过一定时间(时长下面参数决定)，就会被销毁掉

很好理解吧，正常情况下你不干活我也养你，因为我总有用到你的时候，但有时候特殊情况(比如我自己都养不起了)，那你不干活我就要把你干掉了
```

* **int maximumPoolSize   **该线程池中**线程总数最大值**

```
该线程池中线程总数最大值

线程总数 = 核心线程数 + 非核心线程数。核心线程在上面解释过了，这里说下非核心线程：

不是核心线程的线程(别激动，把刀放下...)，其实在上面解释过了
```

* **long keepAliveTime   **该线程池中**非核心线程闲置超时时长**

```
该线程池中非核心线程闲置超时时长

一个非核心线程，如果不干活(闲置状态)的时长超过这个参数所设定的时长，就会被销毁掉

如果设置allowCoreThreadTimeOut = true，则会作用于核心线程
```

* **TimeUnit unit   **keepAliveTime的单位，TimeUnit是一个枚举类型

```
keepAliveTime的单位，TimeUnit是一个枚举类型，其包括：

NANOSECONDS ： 1微毫秒 = 1微秒 / 1000
MICROSECONDS ： 1微秒 = 1毫秒 / 1000
MILLISECONDS ： 1毫秒 = 1秒 /1000
SECONDS ： 秒
MINUTES ： 分
HOURS ： 小时
DAYS ： 天
```

* **BlockingQueue&lt;Runnable&gt;workQueue  **该线程池中的任务队列

```
该线程池中的任务队列：维护着等待执行的Runnable对象

当所有的核心线程都在干活时，新添加的任务会被添加到这个队列中等待处理，如果队列满了，则新建非核心线程执行任务

常用的workQueue类型：

SynchronousQueue：这个队列接收到任务的时候，会直接提交给线程处理，而不保留它，如果所有线程都在工作怎么办？那就新建一个线程来处理这个任务！所以为了保证不出现<线程数达到了maximumPoolSize而不能新建线程>的错误，使用这个类型队列的时候，maximumPoolSize一般指定成Integer.MAX_VALUE，即无限大

LinkedBlockingQueue：这个队列接收到任务的时候，如果当前线程数小于核心线程数，则新建线程(核心线程)处理任务；如果当前线程数等于核心线程数，则进入队列等待。由于这个队列没有最大值限制，即所有超过核心线程数的任务都将被添加到队列中，这也就导致了maximumPoolSize的设定失效，因为总线程数永远不会超过corePoolSize

ArrayBlockingQueue：可以限定队列的长度，接收到任务的时候，如果没有达到corePoolSize的值，则新建线程(核心线程)执行任务，如果达到了，则入队等候，如果队列已满，则新建线程(非核心线程)执行任务，又如果总线程数到了maximumPoolSize，并且队列也满了，则发生错误

DelayedWorkQueue：队列内元素必须实现Delayed接口，这就意味着你传进去的任务必须先实现Delayed接口。这个队列接收到任务时，首先先入队，只有达到了指定的延时时间，才会执行任务
```

* **ThreadFactory threadFactory **创建线程的方式，这是一个接口，你new他的时候需要实现他的`Thread newThread(Runnable r)`

  方法

* **RejectedExecutionHandler handler **抛出异常专用的，比如上面提到的两个错误发生了，就会由这个handler抛出异常

```
这玩意儿就是抛出异常专用的，比如上面提到的两个错误发生了，就会由这个handler抛出异常，你不指定他也有个默认的

抛异常能抛出什么花样来？所以这个星期天不管了，一边去，根本用不上
```

### 1.2.2.ThreadPoolExecutor的策略

上面介绍参数的时候其实已经说到了ThreadPoolExecutor执行的策略，这里给总结一下，当一个任务被添加进线程池时：

1. 线程数量未达到corePoolSize，则新建一个线程\(核心线程\)执行任务
2. 线程数量达到了corePools，则将任务移入队列等待
3. 队列已满，新建线程\(非核心线程\)执行任务
4. 队列已满，总线程数又达到了maximumPoolSize，就会由上面那位星期天\(RejectedExecutionHandler\)抛出异常

## 1.3.四种常见的线程池详解

### 1.3.1.Executors.newCacheThreadPool\(\)：

### 可缓存线程池，先查看池中有没有以前建立的线程，如果有，就直接使用。如果没有，就建一个新的线程加入池中，缓存型池子通常用于执行一些生存期很短的异步型任务

* 底层：返回ThreadPoolExecutor实例，corePoolSize为0；maximumPoolSize为Integer.MAX\_VALUE；keepAliveTime为60L；unit为TimeUnit.SECONDS；workQueue为SynchronousQueue\(同步队列\)

微信截图\_20200428114505.png

* 通俗：当有新任务到来，则插入到SynchronousQueue中，由于SynchronousQueue是同步队列，因此会在池中寻找可用线程来执行，若有可以线程则执行，若没有可用线程则创建一个线程来执行该任务；若池中线程空闲时间超过指定大小，则该线程会被销毁。

* 适用：执行很多短期异步的小程序或者负载较轻的服务器

### 1.3.2.Executors.newFixedThreadPool\(int n\)：创建一个可重用固定个数的线程池，以共享的无界队列方式来运行这些线程。

* 底层：返回ThreadPoolExecutor实例，接收参数为所设定线程数量nThread，corePoolSize为nThread，maximumPoolSize为nThread；keepAliveTime为0L\(不限时\)；unit为：TimeUnit.MILLISECONDS；WorkQueue为：new LinkedBlockingQueue&lt;Runnable&gt;\(\) 无解阻塞队列

微信截图\_20200428114543.png

* 通俗：创建可容纳固定数量线程的池子，每隔线程的存活时间是无限的，当池子满了就不在添加线程了；如果池中的所有线程均在繁忙状态，对于新任务会进入阻塞队列中\(无界的阻塞队列\)

* 适用：执行长期的任务，性能好很多

### 1.3.3.Executors.newScheduledThreadPool\(int n\)：创建一个定长线程池，支持定时及周期性任务执行

* 底层：FinalizableDelegatedExecutorService包装的ThreadPoolExecutor实例，corePoolSize为1；maximumPoolSize为1；keepAliveTime为0L；unit为：TimeUnit.MILLISECONDS；workQueue为：new LinkedBlockingQueue&lt;Runnable&gt;\(\) 无解阻塞队列

微信截图\_20200428114604.png

* 通俗：创建只有一个线程的线程池，且线程的存活时间是无限的；当该线程正繁忙时，对于新任务会进入阻塞队列中\(无界的阻塞队列\)

* 适用：一个任务一个任务执行的场景

### 1.3.4.Executors.newSingleThreadExecutor\(\)：创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序\(FIFO, LIFO, 优先级\)执行。

* 底层：创建ScheduledThreadPoolExecutor实例，corePoolSize为传递来的参数，maximumPoolSize为Integer.MAX\_VALUE；keepAliveTime为0；unit为：TimeUnit.NANOSECONDS；workQueue为：new DelayedWorkQueue\(\) 一个按超时时间升序排序的队列

微信截图\_20200428114616.png

* 通俗：创建一个固定大小的线程池，线程池内线程存活时间无限制，线程池可以支持定时及周期性任务执行，如果所有线程均处于繁忙状态，对于新任务会进入DelayedWorkQueue队列中，这是一种按照超时时间排序的队列结构

* 适用：周期性执行任务的场景

# 2.怎么使用

**源码:**

## 2.1.常用方法

### 1.shutdown方法有2个重载：

**void** shutdown\(\) 启动一次顺序关闭，等待执行以前提交的任务完成，但不接受新任务。

List&lt;Runnable&gt; shutdownNow\(\) 试图立即停止所有正在执行的活动任务，暂停处理正在等待的任务，并返回等待执行的任务列表。

### 2.submit 与 execute

1.submit是ExecutorService中的方法 用以提交一个任务

他的返回值是future对象  可以获取执行结果

&lt;T&gt; Future&lt;T&gt; submit\(Callable&lt;T&gt; task\) 提交一个返回值的任务用于执行，返回一个表示任务的未决结果的 Future。

Future&lt;?&gt; submit\(Runnable task\) 提交一个 Runnable 任务用于执行，并返回一个表示该任务的 Future。

&lt;T&gt; Future&lt;T&gt; submit\(Runnable task, T result\) 提交一个 Runnable 任务用于执行，并返回一个表示该任务的 Future。

2.execute是Executor接口的方法

他虽然也可以像submit那样让一个任务执行  但并不能有返回值

void **execute**\([Runnable](mk:@MSITStore:C:\Users\Administrator\Desktop\JDK1.6 API帮助文档.CHM::/java/lang/Runnable.html) command\)

在未来某个时间执行给定的命令。该命令可能在新的线程、已入池的线程或者正调用的线程中执行，这由 Executor 实现决定。

3.

### Future

Future 表示异步计算的结果。

它提供了检查计算是否完成的方法，以等待计算的完成，并获取计算的结果。

计算完成后只能使用 get 方法来获取结果，如有必要，计算完成前可以阻塞此方法。

取消则由 cancel 方法来执行。还提供了其他方法，以确定任务是正常完成还是被取消了。一旦计算完成，就不能再取消计算。

如果为了可取消性而使用 Future 但又不提供可用的结果，则可以声明 Future&lt;?&gt; 形式类型、并返回 null 作为底层任务的结果。



Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。

必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。　

也就是说Future提供了三种功能：

--判断任务是否完成；

--能够中断任务；

--能够获取任务执行结果。



boolean cancel\(boolean mayInterruptIfRunning\) 试图取消对此任务的执行。

V get\(\) 如有必要，等待计算完成，然后获取其结果。

V get\(long timeout, TimeUnit unit\) 如有必要，最多等待为使计算完成所给定的时间之后，获取其结果（如果结果可用）。

boolean isCancelled\(\) 如果在任务正常完成前将其取消，则返回 true。

boolean isDone\(\) 如果任务已完成，则返回 true。

# 3.总结

## 3.1.阿里巴巴手册

阿里巴巴java开发手册》中指出了线程资源必须通过线程池提供，不允许在应用中自行显示的创建线程，这样一方面是线程的创建更加规范，可以合理控制开辟线程的数量；另一方面线程的细节管理交给线程池处理，优化了资源的开销。而线程池不允许使用Executors去创建，而要通过ThreadPoolExecutor方式，这一方面是由于jdk中Executor框架虽然提供了如newFixedThreadPool\(\)、newSingleThreadExecutor\(\)、newCachedThreadPool\(\)等创建线程池的方法，但都有其局限性，不够灵活【消耗内存等】；另外由于前面几种方法内部也是通过ThreadPoolExecutor方式实现，使用ThreadPoolExecutor有助于大家明确线程池的运行规则，创建符合自己的业务场景需要的线程池，避免资源耗尽的风险，所以阿里巴巴java开发规范线程池首选ThreadPoolExcutor。

```
// 例子
private static final ThreadPoolExecutor THREADPOOL = new ThreadPoolExecutor(2, 4, 3,
        TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(3),
        new ThreadPoolExecutor.DiscardOldestPolicy());

// 其中一个构造方法
private static final ThreadPoolExecutor THREADPOOL =  new ThreadPoolExecutor
(int corePoolSize, 
int maximumPoolSize,
long keepAliveTime, 
TimeUnit unit,
BlockingQueue<Runnable> workQueue,
RejectedExecutionHandler handler)

----------------------------------------------------------------------
        corePoolSize：      线程池维护线程的最少数量 （core : 核心）
        maximumPoolSize：   线程池维护线程的最大数量 
        keepAliveTime：     线程池维护线程所允许的空闲时间
        unit：               线程池维护线程所允许的空闲时间的单位
        workQueue：          线程池所使用的缓冲队列
        handler：            线程池对拒绝任务的处理策略
```

## 3.2.**线程池任务执行流程：**

* 当线程池小于corePoolSize时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程。

* 当线程池达到corePoolSize时，新提交任务将被放入workQueue中，等待线程池中任务调度执行

* 当workQueue已满，且maximumPoolSize&gt;corePoolSize时，新提交任务会创建新线程执行任务

* 当提交任务数超过maximumPoolSize时，新提交任务由RejectedExecutionHandler处理

* 当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程

* 当设置allowCoreThreadTimeOut\(true\)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭

**处理任务的优先级为：**

1.核心线程corePoolSize、任务队列workQueue、最大线程maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。

2.当线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。

3.unit可选的参数为java.util.concurrent.TimeUnit中的几个静态属性：NANOSECONDS、MICROSECONDS、MILLISECONDS、SECONDS。

4.workQueue常用的是：java.util.concurrent.ArrayBlockingQueue

5.**handler有四个选择：**

* ThreadPoolExecutor.AbortPolicy\(\)：     抛出java.util.concurrent.RejectedExecutionException异常

* ThreadPoolExecutor.CallerRunsPolicy\(\):     重试添加当前的任务，他会自动重复调用execute\(\)方法

* ThreadPoolExecutor.DiscardOldestPolicy\(\):     抛弃旧的任务

* ThreadPoolExecutor.DiscardPolicy\(\):     抛弃当前的任务

**线程池的使用场合：     **

```
 （1）单个任务处理的时间比较短；

 （2）需要处理的任务数量大
```

# 4.参考

java 线程池 使用实例：  
[https://www.cnblogs.com/GarfieldEr007/p/10230865.html](https://www.cnblogs.com/GarfieldEr007/p/10230865.html)

Java线程池详解：

[https://www.jianshu.com/p/7726c70cdc40](https://www.jianshu.com/p/7726c70cdc40)

ThreadPoolExcutor线程池使用总结：

[https://blog.csdn.net/weixin\_39352976/article/details/100884832](https://blog.csdn.net/weixin_39352976/article/details/100884832)


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
* **int maximumPoolSize   **该线程池中**线程总数最大值**
* **long keepAliveTime   **该线程池中**非核心线程闲置超时时长**
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

DelayQueue：队列内元素必须实现Delayed接口，这就意味着你传进去的任务必须先实现Delayed接口。这个队列接收到任务时，首先先入队，只有达到了指定的延时时间，才会执行任务
```

* **ThreadFactory threadFactory **创建线程的方式，这是一个接口，你new他的时候需要实现他的`Thread newThread(Runnable r)`

  方法

* **RejectedExecutionHandler handler **抛出异常专用的，比如上面提到的两个错误发生了，就会由这个handler抛出异常

```
这玩意儿就是抛出异常专用的，比如上面提到的两个错误发生了，就会由这个handler抛出异常，你不指定他也有个默认的

抛异常能抛出什么花样来？所以这个星期天不管了，一边去，根本用不上
```

# 2.怎么使用

# 3.参考

java 线程池 使用实例：  
[https://www.cnblogs.com/GarfieldEr007/p/10230865.html](https://www.cnblogs.com/GarfieldEr007/p/10230865.html)

Java线程池详解：

[https://www.jianshu.com/p/7726c70cdc40](https://www.jianshu.com/p/7726c70cdc40)

ThreadPoolExcutor线程池使用总结：

[https://blog.csdn.net/weixin\_39352976/article/details/100884832](https://blog.csdn.net/weixin_39352976/article/details/100884832)


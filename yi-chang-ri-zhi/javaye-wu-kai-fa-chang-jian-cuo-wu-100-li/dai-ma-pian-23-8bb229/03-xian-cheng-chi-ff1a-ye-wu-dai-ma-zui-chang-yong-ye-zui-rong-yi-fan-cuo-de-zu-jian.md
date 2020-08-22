# 03 | 线程池：业务代码最常用也最容易犯错的组件

你好，我是朱晔。今天，我来讲讲使用线程池需要注意的一些问题。

在程序中，我们会用各种池化技术来缓存创建昂贵的对象，比如线程池、连接池、内存池。一般是预先创建一些对象放入池中，使用的时候直接取出使用，用完归还以便复用，还会通过一定的策略调整池中缓存对象的数量，实现池的动态伸缩。

由于线程的创建比较昂贵，随意、没有控制地创建大量线程会造成性能问题，因此短平快的任务一般考虑使用线程池来处理，而不是直接创建线程。

今天，我们就针对线程池这个话题展开讨论，通过三个生产事故，来看看使用线程池应该注意些什么。

## 线程池的声明需要手动进行

Java 中的 Executors 类定义了一些快捷的工具方法，来帮助我们快速创建线程池。《阿里巴巴 Java 开发手册》中提到，禁止使用这些方法来创建线程池，而应该手动 new ThreadPoolExecutor 来创建线程池。这一条规则的背后，是大量血淋淋的生产事故，最典型的就是 newFixedThreadPool 和 newCachedThreadPool，可能因为资源耗尽导致 OOM 问题。

首先，我们来看一下 newFixedThreadPool 为什么可能会出现 OOM 的问题。

我们写一段测试代码，来初始化一个单线程的 FixedThreadPool，循环 1 亿次向线程池提交任务，每个任务都会创建一个比较大的字符串然后休眠一小时：
 


```

@GetMapping("oom1")
public void oom1() throws InterruptedException {

    ThreadPoolExecutor threadPool = (ThreadPoolExecutor) Executors.newFixedThreadPool(1);
    //打印线程池的信息，稍后我会解释这段代码
    printStats(threadPool); 
    for (int i = 0; i < 100000000; i++) {
        threadPool.execute(() -> {
            String payload = IntStream.rangeClosed(1, 1000000)
                    .mapToObj(__ -> "a")
                    .collect(Collectors.joining("")) + UUID.randomUUID().toString();
            try {
                TimeUnit.HOURS.sleep(1);
            } catch (InterruptedException e) {
            }
            log.info(payload);
        });
    }

    threadPool.shutdown();
    threadPool.awaitTermination(1, TimeUnit.HOURS);
}
```
执行程序后不久，日志中就出现了如下 OOM：



```

Exception in thread "http-nio-45678-ClientPoller" java.lang.OutOfMemoryError: GC overhead limit exceeded
```

翻看 newFixedThreadPool 方法的源码不难发现，线程池的工作队列直接 new 了一个 LinkedBlockingQueue，**而默认构造方法的 LinkedBlockingQueue 是一个 Integer.MAX_VALUE 长度的队列，可以认为是无界的：**



```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {



    /**
     * Creates a {@code LinkedBlockingQueue} with a capacity of
     * {@link Integer#MAX_VALUE}.
     */
    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }

}
```

虽然使用 newFixedThreadPool 可以把工作线程控制在固定的数量上，但任务队列是无界的。如果任务较多并且执行较慢的话，队列可能会快速积压，撑爆内存导致 OOM。

我们再把刚才的例子稍微改一下，改为使用 newCachedThreadPool 方法来获得线程池。程序运行不久后，同样看到了如下 OOM 异常：



```

[11:30:30.487] [http-nio-45678-exec-1] [ERROR] [.a.c.c.C.[.[.[/].[dispatcherServlet]:175 ] - Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Handler dispatch failed; nested exception is java.lang.OutOfMemoryError: unable to create new native thread] with root cause
java.lang.OutOfMemoryError: unable to create new native thread 
```

从日志中可以看到，这次 OOM 的原因是无法创建线程，翻看 newCachedThreadPool 的源码可以看到，**这种线程池的最大线程数是 Integer.MAX_VALUE，可以认为是没有上限的，而其工作队列 SynchronousQueue 是一个没有存储空间的阻塞队列。**这意味着，只要有请求到来，就必须找到一条工作线程来处理，如果当前没有空闲的线程就再创建一条新的。



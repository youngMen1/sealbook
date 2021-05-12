# ForkJoinPool原理及使用方式

Fork的是分叉，交叉的意思，Join是合并、聚合的意思。所以Fork Join模式指的就是不停的拆分和聚合的过程。“天下大势，合久必分，分久必合”描述的就是一个不停Fork Join的过程。“分而治之”也是对ForkJoin模式一个描述。我们常见的归并排序算法就是ForkJoin模式的一个具体应用：
88870061-f816aa00-d246-11ea-9a90-744d8f332186.png

ForkJoinPool是JDK1.7出现的一个对于forkjoin模式的实现。通过ForkJoinPool可以轻松做到任务的Fork以及Join操作。同时在JDK1.8时，又使用避免代码伪共享对其性能优化，以及CompletableFuture及集合并行计算等，底层都是基于ForkJoinPool实现。

## 2.使用Demo（可见项目seal-common->seal-features）

接下来使用ForkJoinPool实现一个简单的归并排序算法。

```
 public static void main(String[] args) {
        int[] arr = new int[]{1,3,4,2,5,7,3,8,5,9};
        int[] result = new int[10];
        ForkJoinPool forkJoinPool = ForkJoinPool.commonPool();
        MergerSortTask mergerSortTask = new MergerSortTask(0, arr.length - 1, arr, result);
        forkJoinPool.submit(mergerSortTask);
        System.out.println(Arrays.toString(result));
    }
    static class MergerSortTask extends RecursiveAction {
        private int begin;
        private int end;
        private int[] source;
        private int[] result;
        public MergerSortTask(int begin,int end,int[] source,int[] result) {
            this.begin = begin;
            this.end = end;
            this.source = source;
            this.result = result;
        }
        @Override
        protected void compute() {
            if (end<=begin){
                return;
            }
            int mid = (end - begin)/2;
            MergerSortTask lowerSortTask = new MergerSortTask(begin, mid, source,result);
            MergerSortTask higherSortTask = new MergerSortTask(mid+1, end, source,result);
            invokeAll(lowerSortTask,higherSortTask);
            //merge
            int i = begin;
            int j = mid +1;
            int k = begin;
            while (i <= mid && j <= end)
                result[k++] = source[i] < source[j] ? source[i++] : source[j++];
            while (i <= mid)
                result[k++] = source[i++];
            while (j <= end)
                result[k++] = source[j++];
        }
        public int[] getResult() {
            return result;
        }

    }

```

## 核心原理

ForkJoinPool与ThreadPoolExecutor都是ExecutorService的实现类，同时也都是Doug Lea大神的作品。ForkJoinPool并不是作为ThreadPoolExecutor的优化，两者应该是互补的关系，处理的场景也不同。根本性区别在于ForkJoinPool使用了Fork-Join和Work-Stealing机制。这两个机制保证了ForkJoinPool可以将大任务拆分成多个小任务，使用整个ForkJoinPool的多线程能力参与计算，避免了ThreadPoolExecutor在处理单个大任务造成的线程池吞吐量低问题。但是ForkJoinPool也存在自身的缺点，ForkJoinPool更适合处理可以被Fork的CPU密集型任务，大量能够被快速计算的小任务才能将ForkJoinPool优势发挥的更高。


### work-stealing机制

work-stealing机制，简单来讲，就是一个ForkJoinPool存在多个任务，每个任务都独自包含一个队列，同时还存在一个通用队列。每个线程优先处理自身队列，当自身队列任务为空时，从其他任务或者通用队列里偷取任务执行。

89092715-cc322a80-d3e6-11ea-9171-3227c939f678.jpg

Work-stealing机制是通过空间换取时间的思想。当我们使用ThreadPoolExecutor的时候，如果队列里存在大量任务，同时这些任务的所需要的执行时间又很短。那么线程池里任务将浪费大量的时间在从同步队列里获取任务。work-stealing机制中，每个线程都存在自己的双端队列，线程从自身队头端列获取任务，其他空闲线程从该队列的尾部steal任务。既避免了线程间的竞争，又充分的利用了资源。

### 如何创建一个ForkJoinPool

1、使用Executors工具类

Java8在Exetors工具类中新增了两个工厂方法：


```
// parallelism 定义并行级别
public static ExecutorService newWorkStealingPool(int parallelism);
// 默认JVM可用处理器个数
// Runtime.getRuntime().availableProcessors()
public static ExecutorService newWorkStealingPool();

```

2、使用ForkJoinPool内部提供的初始化commonPool


```
public static ForkJoinPool commonPool();

```
3、使用构造函数（不推荐）


```
public ForkJoinPool(int parallelism,
                    ForkJoinWorkerThreadFactory factory,
                    UncaughtExceptionHandler handler,
                    boolean asyncMode) {
    this(checkParallelism(parallelism),
         checkFactory(factory),
         handler,
         asyncMode ? FIFO_QUEUE : LIFO_QUEUE, // 队列工作模式
         "ForkJoinPool-" + nextPoolId() + "-worker-");
    checkPermission();
}

```

* parallelism：并行级别，默认当前JVM可用处理器个数Runtime.getRuntime().availableProcessors()

* factory：创建自定义工作线程的工厂类



```
public static interface ForkJoinWorkerThreadFactory {
  public ForkJoinWorkerThread newThread(ForkJoinPool pool);
}

```
* handler：用于处理任务线程中的未处理异常。

* asyncMode：用於控制WorkQueue的工作模式。分为FIFO和LIFO两种模式。如果选择asynMode=true则为FIFO，这也是FrokJoinPool默认的实现方式。但是对于递归式的任务执行，LIFO要比FIFO模式更好，性能更高。


### 提交任务至ForkJoinPool

因为ForkJoinPool实现了ExecutorService接口，所以其提交任务的API与ThreadPoolExecutor基本相同


```
// 提交沒有返回值的任务
public void execute(ForkJoinTask<?> task) {
    if (task == null)
        throw new NullPointerException();
    externalPush(task);
}
public void execute(Runnable task) {
    if (task == null)
        throw new NullPointerException();
    ForkJoinTask<?> job;
    if (task instanceof ForkJoinTask<?>) 
        job = (ForkJoinTask<?>) task;
    else
        job = new ForkJoinTask.RunnableExecuteAction(task); 
    externalPush(job);
}
// 提交有返回值的任务
public <T> ForkJoinTask<T> submit(ForkJoinTask<T> task) {
    if (task == null)
        throw new NullPointerException();
    externalPush(task);
    return task;
}
public <T> ForkJoinTask<T> submit(Callable<T> task) {
    ForkJoinTask<T> job = new ForkJoinTask.AdaptedCallable<T>(task);
    externalPush(job);
    return job;
}
public <T> ForkJoinTask<T> submit(Runnable task, T result) {
    ForkJoinTask<T> job = new ForkJoinTask.AdaptedRunnable<T>(task, result);
    externalPush(job);
    return job;
}
public ForkJoinTask<?> submit(Runnable task) {
    if (task == null)
        throw new NullPointerException();
    ForkJoinTask<?> job;
    if (task instanceof ForkJoinTask<?>)
        job = (ForkJoinTask<?>) task;
    else
        job = new ForkJoinTask.AdaptedRunnableAction(task);
    externalPush(job);
    return job;
}
// 同步执行任务
public <T> T invoke(ForkJoinTask<T> task) {
    if (task == null)
        throw new NullPointerException();
    externalPush(task);
    return task.join();
}

```

## ForkJoinTask

提交到ForkJoinPool的任务必须为ForkJoinTask的子类。我们来看一下ForkJoinTask的血缘关系

89102093-72f2e700-d438-11ea-97c0-57230117a90c.png

ForkJoinTask实现了Future接口，所以ForkJoinTask可以以异步的方式获取执行结果。但是我们一般不会直接使用ForkJoinTask而是使用其两个子类RecursiveAction和RecursiveTask,区别仅仅在于该任务是否存在返回值。对于ForkJoinTask来说，最重要的方法就是fork和join，我们来看一下相关方法签名



```
// 将当前任务加入任务队列。如果是ForkJoinWorkerThread则加入工作线程队列，否则加入到通用队列。
public final ForkJoinTask<V> fork() {
        Thread t;
        if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
            ((ForkJoinWorkerThread)t).workQueue.push(this);
        else
            ForkJoinPool.common.externalPush(this);
        return this;
}
// 异步执行当前任务，等待当前任务执行完成并返回，不受线程Interupt状态影响。相似方法quietlyJoin()方法不会拋出异常也不会返回結果。
public final V join() {
        int s;
        if ((s = doJoin() & DONE_MASK) != NORMAL)
            reportException(s);
        return getRawResult();
}
// 同步执行当前任务，知道执行完成。相似方法quietlyInvoke()方法不会拋出异常也不会返回結果，需要使用额外方法处理。
public final V invoke() {
        int s;
        if ((s = doInvoke() & DONE_MASK) != NORMAL)
            reportException(s);
        return getRawResult();
}
//存在编译时异常。在工作线程内部调用get方法与调用join方法相同。在其他线程调用get方法，会受线程Interupt状态影响，需要使用额外方法处理。
public final V get() throws InterruptedException, ExecutionException {
        int s = (Thread.currentThread() instanceof ForkJoinWorkerThread) ?
            doJoin() : externalInterruptibleAwaitDone();
        Throwable ex;
        if ((s &= DONE_MASK) == CANCELLED)
            throw new CancellationException();
        if (s == EXCEPTIONAL && (ex = getThrowableException()) != null)
            throw new ExecutionException(ex);
        return getRawResult();
}
// 执行两个异步任务，第一个任务同步执行，第二个任务异步执行，可以优化性能。
public static void invokeAll(ForkJoinTask<?> t1, ForkJoinTask<?> t2) {
        int s1, s2;
        t2.fork();
        if ((s1 = t1.doInvoke() & DONE_MASK) != NORMAL)
            t1.reportException(s1);
        if ((s2 = t2.doJoin() & DONE_MASK) != NORMAL)
            t2.reportException(s2);
}

```

fork、join和invoke方法的签名都还是比较容易理解，这里要额外介绍一下invokeAll方法。假设，我们提交了一个任务t1到ForkJoinPool, t1会被拆解成两个子任务t2和t3，t2又会被拆分成两个子任务t4和t5，t3又会被拆分成两个子任务t6和t7。我们可以得到如下图的任务关系：

89104282-bf472280-d44a-11ea-827d-669297983457.png

如果我们使用最浅显易懂的先fork再join的方式，可以得到如下的任务入队和出队流程：

89104301-f7e6fc00-d44a-11ea-93de-c507c63db404.png

可以看出，最先入队的任务t2虽然是最先执行，但是因为默认的FIFO方式，最先执行完成却是t3, t2任务等待任务执行完成用了很多步骤。
如果使用上文demo使用的invokeAll方法，可以得到如下的任务入队和出队流程：

89104485-48ab2480-d44c-11ea-8fb6-b470c61046d9.png
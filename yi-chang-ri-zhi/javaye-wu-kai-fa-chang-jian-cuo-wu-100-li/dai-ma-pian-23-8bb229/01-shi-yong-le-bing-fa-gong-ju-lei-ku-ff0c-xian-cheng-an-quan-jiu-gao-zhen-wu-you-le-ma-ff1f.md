# 01 | 使用了并发工具类库，线程安全就高枕无忧了吗？

你好，我是朱晔。作为课程的第一讲，我今天要和你聊聊使用并发工具类库相关的话题。在代码审核讨论的时候，我们有时会听到有关线程安全和并发工具的一些片面的观点和结论，比如“把 HashMap 改为 ConcurrentHashMap，就可以解决并发问题了呀”“要不我们试试无锁的 CopyOnWriteArrayList 吧，性能更好”。事实上，这些说法都不太准确。

的确，为了方便开发者进行多线程编程，现代编程语言会提供各种并发工具类。但如果我们没有充分了解它们的使用场景、解决的问题，以及最佳实践的话，盲目使用就可能会导致一些坑，小则损失性能，大则无法确保多线程情况下业务逻辑的正确性。

我需要先说明下，这里的并发工具类是指用来解决多线程环境下并发问题的工具类库。一般而言并发工具包括同步器和容器两大类，业务代码中使用并发容器的情况会多一些，我今天分享的例子也会侧重并发容器。

接下来，我们就看看在使用并发工具时，最常遇到哪些坑，以及如何解决、避免这些坑吧。

## 没有意识到线程重用导致用户信息错乱的 Bug

之前有业务同学和我反馈，在生产上遇到一个诡异的问题，有时获取到的用户信息是别人的。查看代码后，我发现他使用了 ThreadLocal 来缓存获取到的用户信息。我们知道，ThreadLocal 适用于变量在线程间隔离，而在方法或类间共享的场景。如果用户信息的获取比较昂贵（比如从数据库查询用户信息），那么在 ThreadLocal 中缓存数据是比较合适的做法。但，这么做为什么会出现用户信息错乱的 Bug 呢？

我们看一个具体的案例吧。

使用 Spring Boot 创建一个 Web 应用程序，使用 ThreadLocal 存放一个 Integer 的值，来暂且代表需要在线程中保存的用户信息，这个值初始是 null。在业务逻辑中，我先从 ThreadLocal 获取一次值，然后把外部传入的参数设置到 ThreadLocal 中，来模拟从当前上下文获取到用户信息的逻辑，随后再获取一次值，最后输出两次获得的值和线程名称。

```

private static final ThreadLocal<Integer> currentUser = ThreadLocal.withInitial(() -> null);


@GetMapping("wrong")
public Map wrong(@RequestParam("userId") Integer userId) {
    //设置用户信息之前先查询一次ThreadLocal中的用户信息
    String before  = Thread.currentThread().getName() + ":" + currentUser.get();
    //设置用户信息到ThreadLocal
    currentUser.set(userId);
    //设置用户信息之后再查询一次ThreadLocal中的用户信息
    String after  = Thread.currentThread().getName() + ":" + currentUser.get();
    //汇总输出两次查询结果
    Map result = new HashMap();
    result.put("before", before);
    result.put("after", after);
    return result;
}

```

按理说，在设置用户信息之前第一次获取的值始终应该是 null，但我们要意识到，程序运行在 Tomcat 中，执行程序的线程是 Tomcat 的工作线程，而 Tomcat 的工作线程是基于线程池的。


**顾名思义，线程池会重用固定的几个线程，一旦线程重用，那么很可能首次从 ThreadLocal 获取的值是之前其他用户的请求遗留的值。这时，ThreadLocal 中的用户信息就是其他用户的信息。
**

为了更快地重现这个问题，我在配置文件中设置一下 Tomcat 的参数，把工作线程池最大线程数设置为 1，这样始终是同一个线程在处理请求：


```

server.tomcat.max-threads=1
```
运行程序后先让用户 1 来请求接口，可以看到第一和第二次获取到用户 ID 分别是 null 和 1，符合预期：
4b8f38415d03423132c7a3608ebe2430.png

随后用户 2 来请求接口，这次就出现了 Bug，第一和第二次获取到用户 ID 分别是 1 和 2，显然第一次获取到了用户 1 的信息，原因就是 Tomcat 的线程池重用了线程。从图中可以看到，两次请求的线程都是同一个线程：http-nio-8080-exec-1。

a9ccd42716d807687b3acff9a0baf2db.png

这个例子告诉我们，在写业务代码时，首先要理解代码会跑在什么线程上：

* 我们可能会抱怨学多线程没用，因为代码里没有开启使用多线程。但其实，可能只是我们没有意识到，在 Tomcat 这种 Web 服务器下跑的业务代码，本来就运行在一个多线程环境（否则接口也不可能支持这么高的并发），并不能认为没有显式开启多线程就不会有线程安全问题。

* 因为线程的创建比较昂贵，所以 Web 服务器往往会使用线程池来处理请求，这就意味着线程会被重用。这时，使用类似 ThreadLocal 工具来存放一些数据时，需要特别注意在代码运行完后，显式地去清空设置的数据。如果在代码中使用了自定义的线程池，也同样会遇到这个问题。

理解了这个知识点后，我们修正这段代码的方案是，在代码的 finally 代码块中，显式清除 ThreadLocal 中的数据。这样一来，新的请求过来即使使用了之前的线程也不会获取到错误的用户信息了。修正后的代码如下：



```

@GetMapping("right")
public Map right(@RequestParam("userId") Integer userId) {
    String before  = Thread.currentThread().getName() + ":" + currentUser.get();
    currentUser.set(userId);
    try {
        String after = Thread.currentThread().getName() + ":" + currentUser.get();
        Map result = new HashMap();
        result.put("before", before);
        result.put("after", after);
        return result;
    } finally {
        //在finally代码块中删除ThreadLocal中的数据，确保数据不串
        currentUser.remove();
    }
}
```
重新运行程序可以验证，再也不会出现第一次查询用户信息查询到之前用户请求的 Bug：

0dfe40fca441b58d491fc799d120a7cc.png

ThreadLocal 是利用独占资源的方式，来解决线程安全问题，那如果我们确实需要有资源在线程之间共享，应该怎么办呢？这时，我们可能就需要用到线程安全的容器了。

## 使用了线程安全的并发工具，并不代表解决了所有线程安全问题

JDK 1.5 后推出的 ConcurrentHashMap，是一个高性能的线程安全的哈希表容器。“线程安全”这四个字特别容易让人误解，**因为 ConcurrentHashMap 只能保证提供的原子性读写操作是线程安全的。**

我在相当多的业务代码中看到过这个误区，比如下面这个场景。有一个含 900 个元素的 Map，现在再补充 100 个元素进去，这个补充操作由 10 个线程并发进行。开发人员误以为使用了 ConcurrentHashMap 就不会有线程安全问题，于是不加思索地写出了下面的代码：在每一个线程的代码逻辑中先通过 size 方法拿到当前元素数量，计算 ConcurrentHashMap 目前还需要补充多少元素，并在日志中输出了这个值，然后通过 putAll 方法把缺少的元素添加进去。为方便观察问题，我们输出了这个 Map 一开始和最后的元素个数。


```

//线程个数
private static int THREAD_COUNT = 10;
//总元素数量
private static int ITEM_COUNT = 1000;

//帮助方法，用来获得一个指定元素数量模拟数据的ConcurrentHashMap
private ConcurrentHashMap<String, Long> getData(int count) {
    return LongStream.rangeClosed(1, count)
            .boxed()
            .collect(Collectors.toConcurrentMap(i -> UUID.randomUUID().toString(), Function.identity(),
                    (o1, o2) -> o1, ConcurrentHashMap::new));
}

@GetMapping("wrong")
public String wrong() throws InterruptedException {
    ConcurrentHashMap<String, Long> concurrentHashMap = getData(ITEM_COUNT - 100);
    //初始900个元素
    log.info("init size:{}", concurrentHashMap.size());

    ForkJoinPool forkJoinPool = new ForkJoinPool(THREAD_COUNT);
    //使用线程池并发处理逻辑
    forkJoinPool.execute(() -> IntStream.rangeClosed(1, 10).parallel().forEach(i -> {
        //查询还需要补充多少个元素
        int gap = ITEM_COUNT - concurrentHashMap.size();
        log.info("gap size:{}", gap);
        //补充元素
        concurrentHashMap.putAll(getData(gap));
    }));
    //等待所有任务完成
    forkJoinPool.shutdown();
    forkJoinPool.awaitTermination(1, TimeUnit.HOURS);
    //最后元素个数会是1000吗？
    log.info("finish size:{}", concurrentHashMap.size());
    return "OK";
}

```

重新调用接口，程序的日志输出结果符合预期：
1151b5b87f27073725060b76c56d95b8.png

可以看到，只有一个线程查询到了需要补 100 个元素，其他 9 个线程查询到不需要补元素，最后 Map 大小为 1000。


到了这里，你可能又要问了，使用 ConcurrentHashMap 全程加锁，还不如使用普通的 HashMap 呢。

其实不完全是这样。

**ConcurrentHashMap 提供了一些原子性的简单复合逻辑方法，用好这些方法就可以发挥其威力。**这就引申出代码中常见的另一个问题：在使用一些类库提供的高级工具类时，开发人员可能还是按照旧的方式去使用这些新类，因为没有使用其特性，所以无法发挥其威力。

**注意：**可见原子性才是并发的关键，原子性是为了维护状态的一致性，进一步又需要隔离星的保证。 总的来说就是一个非持久化的事物。所以利用ConcurrentHashmap主要利用其原子性的方法。

没有充分了解并发工具的特性，从而无法发挥其威力

## 没有充分了解并发工具的特性，从而无法发挥其威力

我们来看一个使用 Map 来统计 Key 出现次数的场景吧，这个逻辑在业务代码中非常常见。
* 使用 ConcurrentHashMap 来统计，Key 的范围是 10。
* 使用最多 10 个并发，循环操作 1000 万次，每次操作累加随机的 Key。
* 如果 Key 不存在的话，首次设置值为 1。

代码如下：


```

//循环次数
private static int LOOP_COUNT = 10000000;
//线程数量
private static int THREAD_COUNT = 10;
//元素数量
private static int ITEM_COUNT = 10;
private Map<String, Long> normaluse() throws InterruptedException {
    ConcurrentHashMap<String, Long> freqs = new ConcurrentHashMap<>(ITEM_COUNT);
    ForkJoinPool forkJoinPool = new ForkJoinPool(THREAD_COUNT);
    forkJoinPool.execute(() -> IntStream.rangeClosed(1, LOOP_COUNT).parallel().forEach(i -> {
        //获得一个随机的Key
        String key = "item" + ThreadLocalRandom.current().nextInt(ITEM_COUNT);
                synchronized (freqs) {      
                    if (freqs.containsKey(key)) {
                        //Key存在则+1
                        freqs.put(key, freqs.get(key) + 1);
                    } else {
                        //Key不存在则初始化为1
                        freqs.put(key, 1L);
                    }
                }
            }
    ));
    forkJoinPool.shutdown();
    forkJoinPool.awaitTermination(1, TimeUnit.HOURS);
    return freqs;
}
```
我们吸取之前的教训，直接通过锁的方式锁住 Map，然后做判断、读取现在的累计值、加 1、保存累加后值的逻辑。这段代码在功能上没有问题，但无法充分发挥 ConcurrentHashMap 的威力，改进后的代码如下：


```

private Map<String, Long> gooduse() throws InterruptedException {
    ConcurrentHashMap<String, LongAdder> freqs = new ConcurrentHashMap<>(ITEM_COUNT);
    ForkJoinPool forkJoinPool = new ForkJoinPool(THREAD_COUNT);
    forkJoinPool.execute(() -> IntStream.rangeClosed(1, LOOP_COUNT).parallel().forEach(i -> {
        String key = "item" + ThreadLocalRandom.current().nextInt(ITEM_COUNT);
                //利用computeIfAbsent()方法来实例化LongAdder，然后利用LongAdder来进行线程安全计数
                freqs.computeIfAbsent(key, k -> new LongAdder()).increment();
            }
    ));
    forkJoinPool.shutdown();
    forkJoinPool.awaitTermination(1, TimeUnit.HOURS);
    //因为我们的Value是LongAdder而不是Long，所以需要做一次转换才能返回
    return freqs.entrySet().stream()
            .collect(Collectors.toMap(
                    e -> e.getKey(),
                    e -> e.getValue().longValue())
            );
}
```
在这段改进后的代码中，我们巧妙利用了下面两点：
* 使用 ConcurrentHashMap 的原子性方法 computeIfAbsent 来做复合逻辑操作，判断 Key 是否存在 Value，如果不存在则把 Lambda 表达式运行后的结果放入 Map 作为 Value，也就是新创建一个 LongAdder 对象，最后返回 Value。
* 由于 computeIfAbsent 方法返回的 Value 是 LongAdder，是一个线程安全的累加器，因此可以直接调用其 increment 方法进行累加。



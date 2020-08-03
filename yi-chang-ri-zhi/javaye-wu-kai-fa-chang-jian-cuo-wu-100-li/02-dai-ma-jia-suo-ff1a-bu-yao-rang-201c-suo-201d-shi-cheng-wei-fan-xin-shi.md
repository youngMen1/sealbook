# 1.代码加锁：不要让“锁”事成为烦心事

在上一讲中，我与你介绍了使用并发容器等工具解决线程安全的误区。今天，我们来看看解决线程安全问题的另一种重要手段——锁，在使用上比较容易犯哪些错。

我先和你分享一个有趣的案例吧。有一天，一位同学在群里说“见鬼了，疑似遇到了一个 JVM 的 Bug”，我们都很好奇是什么 Bug。

于是，他贴出了这样一段代码：在一个类里有两个 int 类型的字段 a 和 b，有一个 add 方法循环 1 万次对 a 和 b 进行 ++ 操作，有另一个 compare 方法，同样循环 1 万次判断 a 是否小于 b，条件成立就打印 a 和 b 的值，并判断 a>b 是否成立。

```
@Slf4j
public class Interesting {

    volatile int a = 1;
    volatile int b = 1;

    public void add() {
        log.info("add start");
        for (int i = 0; i < 10000; i++) {
            a++;
            b++;
        }
        log.info("add done");
    }

    public void compare() {
        log.info("compare start");
        for (int i = 0; i < 10000; i++) {
            //a始终等于b吗？
            if (a < b) {
                log.info("a:{},b:{},{}", a, b, a > b);
                //最后的a>b应该始终是false吗？
            }
        }
        log.info("compare done");
    }
}
```

他起了两个线程来分别执行 add 和 compare 方法：


```

Interesting interesting = new Interesting();
new Thread(() -> interesting.add()).start();
new Thread(() -> interesting.compare()).start();
```

按道理，a 和 b 同样进行累加操作，应该始终相等，compare 中的第一次判断应该始终不会成立，不会输出任何日志。但，执行代码后发现不但输出了日志，而且更诡异的是，compare 方法在判断 ab 也成立：
![](/static/image/9ec61aada64ac6d38681dd199c0ee61d.png)
群里一位同学看到这个问题笑了，说：“这哪是 JVM 的 Bug，分明是线程安全问题嘛。很明显，你这是在操作两个字段 a 和 b，有线程安全问题，应该为 add 方法加上锁，确保 a 和 b 的 ++ 是原子性的，就不会错乱了。”随后，他为 add 方法加上了锁：


```

public synchronized void add()
```
但，加锁后问题并没有解决。

我们来仔细想一下，为什么锁可以解决线程安全问题呢。因为只有一个线程可以拿到锁，所以加锁后的代码中的资源操作是线程安全的。但是，**这个案例中的 add 方法始终只有一个线程在操作，显然只为 add 方法加锁是没用的。**

之所以出现这种错乱，是因为两个线程是交错执行 add 和 compare 方法中的业务逻辑，而且这些业务逻辑不是原子性的：a++ 和 b++ 操作中可以穿插在 compare 方法的比较代码中；更需要注意的是，a<b 这种比较操作在字节码层面是加载 a、加载 b 和比较三步，代码虽然是一行但也不是原子性的。

所以，正确的做法应该是，为 add 和 compare 都加上方法锁，确保 add 方法执行时，compare 无法读取 a 和 b：

```
public synchronized void add()
public synchronized void compare()
```

所以，使用锁解决问题之前一定要理清楚，我们要保护的是什么逻辑，多线程执行的情况又是怎样的。

### 加锁前要清楚锁和被保护的对象是不是一个层面的

除了没有分析清线程、业务逻辑和锁三者之间的关系随意添加无效的方法锁外，还有一种比较常见的错误是，没有理清楚锁和要保护的对象是否是一个层面的。


我们知道**静态字段属于类，类级别的锁才能保护；而非静态字段属于类实例，实例级别的锁就可以保护。**

先看看这段代码有什么问题：在类 Data 中定义了一个静态的 int 字段 counter 和一个非静态的 wrong 方法，实现 counter 字段的累加操作。

```
class Data {
    @Getter
    private static int counter = 0;
    
    public static int reset() {
        counter = 0;
        return counter;
    }

    public synchronized void wrong() {
        counter++;
    }
}
```

写一段代码测试下：

```
@GetMapping("wrong")
public int wrong(@RequestParam(value = "count", defaultValue = "1000000") int count) {
    Data.reset();
    //多线程循环一定次数调用Data类不同实例的wrong方法
    IntStream.rangeClosed(1, count).parallel().forEach(i -> new Data().wrong());
    return Data.getCounter();
}
```

因为默认运行 100 万次，所以执行后应该输出 100 万，但页面输出的是 639242：

777f520e9d0be89b66e814d3e7c1a30b.png

我们来分析下为什么会出现这个问题吧。

在非静态的 wrong 方法上加锁，只能确保多个线程无法执行同一个实例的 wrong 方法，却不能保证不会执行不同实例的 wrong 方法。而静态的 counter 在多个实例中共享，所以必然会出现线程安全问题。

理清思路后，修正方法就很清晰了：同样在类中定义一个 Object 类型的静态字段，在操作 counter 之前对这个字段加锁。

```
class Data {
    @Getter
    private static int counter = 0;
    private static Object locker = new Object();

    public void right() {
        synchronized (locker) {
            counter++;
        }
    }
}
```

你可能要问了，把 wrong 方法定义为静态不就可以了，这个时候锁是类级别的。可以是可以，但我们不可能为了解决线程安全问题改变代码结构，把实例方法改为静态方法。

感兴趣的同学还可以从字节码以及 JVM 的层面继续探索一下，代码块级别的 synchronized 和方法上标记 synchronized 关键字，在实现上有什么区别。

## 加锁要考虑锁的粒度和场景问题
在方法上加 synchronized 关键字实现加锁确实简单，也因此我曾看到一些业务代码中几乎所有方法都加了 synchronized，但这种滥用 synchronized 的做法：

* 一是，没必要。通常情况下 60% 的业务代码是三层架构，数据经过无状态的 Controller、Service、Repository 流转到数据库，没必要使用 synchronized 来保护什么数据。

* 二是，可能会极大地降低性能。使用 Spring 框架时，默认情况下 Controller、Service、Repository 是单例的，加上 synchronized 会导致整个程序几乎就只能支持单线程，造成极大的性能问题。

**
即使我们确实有一些共享资源需要保护，也要尽可能降低锁的粒度，仅对必要的代码块甚至是需要保护的资源本身加锁。**

比如，在业务代码中，有一个 ArrayList 因为会被多个线程操作而需要保护，又有一段比较耗时的操作（代码中的 slow 方法）不涉及线程安全问题，应该如何加锁呢？


错误的做法是，给整段业务逻辑加锁，把 slow 方法和操作 ArrayList 的代码同时纳入 synchronized 代码块；更合适的做法是，把加锁的粒度降到最低，只在操作 ArrayList 的时候给这个 ArrayList 加锁。



```

private List<Integer> data = new ArrayList<>();

//不涉及共享资源的慢方法
private void slow() {
    try {
        TimeUnit.MILLISECONDS.sleep(10);
    } catch (InterruptedException e) {
    }
}

//错误的加锁方法
@GetMapping("wrong")
public int wrong() {
    long begin = System.currentTimeMillis();
    IntStream.rangeClosed(1, 1000).parallel().forEach(i -> {
        //加锁粒度太粗了
        synchronized (this) {
            slow();
            data.add(i);
        }
    });
    log.info("took:{}", System.currentTimeMillis() - begin);
    return data.size();
}

//正确的加锁方法
@GetMapping("right")
public int right() {
    long begin = System.currentTimeMillis();
    IntStream.rangeClosed(1, 1000).parallel().forEach(i -> {
        slow();
        //只对List加锁
        synchronized (data) {
            data.add(i);
        }
    });
    log.info("took:{}", System.currentTimeMillis() - begin);
    return data.size();
}
```

执行这段代码，同样是 1000 次业务操作，正确加锁的版本耗时 1.4 秒，而对整个业务逻辑加锁的话耗时 11 秒。

1cb278c010719ee00d988dbb2a42c543.png

**如果精细化考虑了锁应用范围后，性能还无法满足需求的话，我们就要考虑另一个维度的粒度问题了，即：区分读写场景以及资源的访问冲突，考虑使用悲观方式的锁还是乐观方式的锁。**


* 一般业务代码中，很少需要进一步考虑这两种更细粒度的锁，所以我只和你分享几个大概的结论，你可以根据自己的需求来考虑是否有必要进一步优化：

* 对于读写比例差异明显的场景，考虑使用 ReentrantReadWriteLock 细化区分读写锁，来提高性能。
如果你的 JDK 版本高于 1.8、共享资源的冲突概率也没那么大的话，考虑使用 StampedLock 的乐观读的特性，进一步提高性能。
* JDK 里 ReentrantLock 和 ReentrantReadWriteLock 都提供了公平锁的版本，在没有明确需求的情况下不要轻易开启公平锁特性，在任务很轻的情况下开启公平锁可能会让性能下降上百倍。

## 多把锁要小心死锁问题

# 2.总结
## 2.1.高质量问题
1.加锁解锁没有配对可以用一些代码质量工具协助排插，如Sonar，集成到ide和代码仓库，在编码阶段发现，加上超时自动释放，避免长期占有锁
2.锁超时自动释放导致重复执行的话，可以用锁续期，如redisson的watchdog；或者保证业务的幂等性，重复执行也没问题。

**回复：**这个回答太赞了！

2.超时自动释放锁后怎么避免重复逻辑好难，面试曾被卡，求解。。。

**回复：**
有两个方面：
1. 避免超时，单独开一个线程给锁延长有效期。比如设置锁有效期30s，有个线程每隔10s重新设置下锁的有效期。 
2. 避免重复，业务上增加一个标记是否被处理的字段。或者开一张新表，保存已经处理过的流水号。

3.关于锁过期问题。以前做redis分布式锁的时候一直在思考这个问题。当时觉得就是尽量让锁过期时间比程序执行之间略长一些，以保证加锁区域代码能尽量执行完成。看到老师给其他同学评论说可以用另外一个线程去不断重置锁时间，这里有我理解是针对像redis这种利用setnx实现的分布式锁可以这么解决。那还有其他场景吗？

**回复：**
就是锁续期解决 可以看一下redisson实现
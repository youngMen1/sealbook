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
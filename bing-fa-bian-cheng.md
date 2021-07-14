# CompletableFuture异步编程

用多线程优化性能，其实不过就是将串行操作变成并行操作。如果仔细观察，你还会发现在串行转换成并行的过程中，一定会涉及到异步化，例如下面的示例代码，现在是串行的，为了提升性能，我们得把它们并行化，那具体实施起来该怎么做呢？

```
//以下两个方法都是耗时操作
doBizA();
doBizB();
```

还是挺简单的，就像下面代码中这样，创建两个子线程去执行就可以了。你会发现下面的并行方案，主线程无需等待 doBizA\(\) 和 doBizB\(\) 的执行结果，也就是说 doBizA\(\) 和 doBizB\(\) 两个操作已经被异步化了。

```
new Thread(()->doBizA())
  .start();
new Thread(()->doBizB())
  .start();
```

异步化，是并行方案得以实施的基础，更深入地讲其实就是：利用多线程优化性能这个核心方案得以实施的基础。看到这里，相信你应该就能理解异步编程最近几年为什么会大火了，因为优化性能是互联网大厂的一个核心需求啊。Java 在 1.8 版本提供了 CompletableFuture 来支持异步编程，CompletableFuture 有可能是你见过的最复杂的工具类了，不过功能也着实让人感到震撼。

## CompletableFuture 的核心优势

为了领略 CompletableFuture 异步编程的优势，这里我们用 CompletableFuture 重新实现前面曾提及的烧水泡茶程序。首先还是需要先完成分工方案，在下面的程序中，  
我们分了 3 个任务：  
任务 1 负责洗水壶、烧开水，  
任务 2 负责洗茶壶、洗茶杯和拿茶叶，  
任务 3 负责泡茶。  
其中任务 3 要等待任务 1 和任务 2 都完成后才能开始。这个分工如下图所示。  
![](/static/image/微信图片_20210714205644.jpg)

下面是代码实现，你先略过 runAsync\(\)、supplyAsync\(\)、thenCombine\(\) 这些不太熟悉的方法，从大局上看，你会发现：  
无需手工维护线程，没有繁琐的手工维护线程的工作，给任务分配线程的工作也不需要我们关注；  
语义更清晰，例如 f3 = f1.thenCombine\(f2, \(\)-&gt;{}\) 能够清晰地表述“任务 3 要等待任务 1 和任务 2 都完成后才能开始”；

```
//任务1：洗水壶->烧开水
CompletableFuture<Void> f1 = 
  CompletableFuture.runAsync(()->{
  System.out.println("T1:洗水壶...");
  sleep(1, TimeUnit.SECONDS);

  System.out.println("T1:烧开水...");
  sleep(15, TimeUnit.SECONDS);
});
//任务2：洗茶壶->洗茶杯->拿茶叶
CompletableFuture<String> f2 = 
  CompletableFuture.supplyAsync(()->{
  System.out.println("T2:洗茶壶...");
  sleep(1, TimeUnit.SECONDS);

  System.out.println("T2:洗茶杯...");
  sleep(2, TimeUnit.SECONDS);

  System.out.println("T2:拿茶叶...");
  sleep(1, TimeUnit.SECONDS);
  return "龙井";
});
//任务3：任务1和任务2完成后执行：泡茶
CompletableFuture<String> f3 = 
  f1.thenCombine(f2, (__, tf)->{
    System.out.println("T1:拿到茶叶:" + tf);
    System.out.println("T1:泡茶...");
    return "上茶:" + tf;
  });
//等待任务3执行结果
System.out.println(f3.join());

void sleep(int t, TimeUnit u) {
  try {
    u.sleep(t);
  }catch(InterruptedException e){}
}
// 一次执行结果：
T1:洗水壶...
T2:洗茶壶...
T1:烧开水...
T2:洗茶杯...
T2:拿茶叶...
T1:拿到茶叶:龙井
T1:泡茶...
上茶:龙井
```

## 创建 CompletableFuture 对象

创建 CompletableFuture 对象主要靠下面代码中展示的这 4 个静态方法，我们先看前两个。在烧水泡茶的例子中，我们已经使用了runAsync\(Runnable runnable\)和`supplyAsync(Supplier<U> supplier)`，它们之间的区别是：Runnable 接口的 run\(\) 方法没有返回值，而 Supplier 接口的 get\(\) 方法是有返回值的。  
前两个方法和后两个方法的区别在于：后两个方法可以指定线程池参数。  
默认情况下 CompletableFuture 会使用公共的 ForkJoinPool 线程池，这个线程池默认创建的线程数是 CPU 的核数（也可以通过 JVM option:-Djava.util.concurrent.ForkJoinPool.common.parallelism 来设置 ForkJoinPool 线程池的线程数）。如果所有 CompletableFuture 共享一个线程池，那么一旦有任务执行一些很慢的 I/O 操作，就会导致线程池中所有线程都阻塞在 I/O 操作上，从而造成线程饥饿，进而影响整个系统的性能。所以，强烈建议你要根据不同的业务类型创建不同的线程池，以避免互相干扰。

```
//使用默认线程池
static CompletableFuture<Void> 
  runAsync(Runnable runnable)
static <U> CompletableFuture<U> 
  supplyAsync(Supplier<U> supplier)
//可以指定线程池  
static CompletableFuture<Void> 
  runAsync(Runnable runnable, Executor executor)
static <U> CompletableFuture<U> 
  supplyAsync(Supplier<U> supplier, Executor executor)
```

创建完 CompletableFuture 对象之后，会自动地异步执行 runnable.run\(\) 方法或者 supplier.get\(\) 方法，对于一个异步操作，你需要关注两个问题：一个是异步操作什么时候结束，另一个是如何获取异步操作的执行结果。因为 CompletableFuture 类实现了 Future 接口，所以这两个问题你都可以通过 Future 接口来解决。另外，CompletableFuture 类还实现了 CompletionStage 接口，这个接口内容实在是太丰富了，在 1.8 版本里有 40 个方法，这些方法我们该如何理解呢？

## 例子
#### 烧水泡茶的例子
runAsync(Runnable runnable)--->run()方法没有返回值
supplyAsync(Supplier<U> supplier)--->调用方通过join或者get就能取到该CompletableFuture的result字段的值。（
1.join()方法抛出的是uncheck异常（即未经检查的异常),不会强制开发者抛出，
2.get()方法抛出的是经过检查的异常，ExecutionException, InterruptedException 需要用户手动处理（抛出或者 try catch））
thenCombine()--->thenCombine会在两个任务都执行完成后，把两个任务的结果合并。
```
    /**
     * 我们分了3个任务:
     * 任务1-> 负责洗水壶、烧开水
     * 任务2-> 负责洗茶壶、洗茶杯和拿茶叶
     * 任务3-> 要等待任务 1 和任务 2 都完成后才能开始。
     */
    private static void createCompletableFuture() {
        // 任务1：洗水壶->烧开水
        CompletableFuture<Void> f1 =
                CompletableFuture.runAsync(() -> {
                    System.out.println("T1:洗水壶...");
                    sleep(1, TimeUnit.SECONDS);

                    System.out.println("T1:烧开水...");
                    sleep(15, TimeUnit.SECONDS);
                });
        // 任务2：洗茶壶->洗茶杯->拿茶叶
        CompletableFuture<String> f2 =
                CompletableFuture.supplyAsync(() -> {
                    System.out.println("T2:洗茶壶...");
                    sleep(1, TimeUnit.SECONDS);

                    System.out.println("T2:洗茶杯...");
                    sleep(2, TimeUnit.SECONDS);

                    System.out.println("T2:拿茶叶...");
                    sleep(1, TimeUnit.SECONDS);
                    return "龙井";
                });
        // 任务3：任务1和任务2完成后执行：泡茶
        CompletableFuture<String> f3 =
                f1.thenCombine(f2, (result1, result2) -> {
                    System.out.println("T1:拿到茶叶:" + result2);
                    System.out.println("T1:泡茶...");
                    return "上茶:" + result2;
                });
        // 等待任务3执行结果
        System.out.println(f3.join());

        //                一次执行结果：
        //                T1:洗水壶...
        //                T2:洗茶壶...
        //                T1:烧开水...
        //                T2:洗茶杯...
        //                T2:拿茶叶...
        //                T1:拿到茶叶:龙井
        //                T1:泡茶...
        //                上茶:龙井
    }

    private static void sleep(int t, TimeUnit u) {
        try {
            u.sleep(t);
        } catch (InterruptedException e) {
            System.out.println(e);
        }
    }
```

#### thenApply()例子
还是原来的CompletableFuture，相当于将CompletableFuture<T> 转换成CompletableFuture<U>,只是泛型从Student转换成Person。
```
    /**
     * thenApply()例子
     */
    private static void completableFutureExample() {
        CompletableFuture<Person> future1 = CompletableFuture.supplyAsync(() -> {
            Student student = new Student();
            student.setName("张三");
            student.setSex("男");
            student.setMoney(1.1D);
            return student;
        }).thenApply(student -> {
            Person person = new Person();
            person.setName(student.getName());
            person.setAge(student.getSex().equals("男") ? "0" : "1");
            person.setCity(new City("11"));
            person.setHeight(175);
            return person;
        });
        Person person = future1.join();
        System.out.println(person.toString());
    }
```
#### thenCompose()例子
用来连接两个CompletableFuture，是生成一个新的CompletableFuture。
```


    /**
     * thenCompose()例子
     */
    private static void completableFutureExample2() {
        CompletableFuture<Person> future1 = CompletableFuture.supplyAsync(() -> {
            Student student = new Student();
            student.setName("张三");
            student.setSex("男");
            student.setMoney(1.1D);
            return student;
        }).thenCompose(student -> CompletableFuture.supplyAsync(()->{
            // 注意这里用到了上个线程的返回值student
            Person person = new Person();
            person.setName(student.getName());
            person.setAge(student.getSex().equals("男") ? "0" : "1");
            person.setCity(new City("11"));
            person.setHeight(175);
            return person;
        }));
        Person person = future1.join();
        System.out.println(person.toString());
    }
```

#### thenApplyAsync()方法例子
链式编程
```


    /**
     * thenApplyAsync()方法例子链式编程
     * [Student(name=张三, sex=男, money=0.0), Student(name=李四, sex=男, money=0.0), Student(name=王五, sex=男, money=0.0)]
     */
    private static void completableFutureExample3() {
        CompletableFuture<List<Student>> future = CompletableFuture.supplyAsync(() -> {
            List<Student> list = new ArrayList<>();
            Student student = new Student();
            student.setName("张三");
            student.setSex("男");
            student.setMoney(0);
            list.add(student);
            return list;
        }).thenApplyAsync(list->{
            Student student = new Student();
            student.setName("李四");
            student.setSex("男");
            student.setMoney(0);
            list.add(student);
            return list;
        }).thenApplyAsync(list->{
            Student student = new Student();
            student.setName("王五");
            student.setSex("男");
            student.setMoney(0);
            list.add(student);
            return list;
        });
        List<Student> result = future.join();
        System.out.println(result.toString());
        
    }
```




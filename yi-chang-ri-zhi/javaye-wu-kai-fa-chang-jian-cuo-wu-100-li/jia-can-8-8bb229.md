# 31 | 加餐1：带你吃透课程中Java 8的那些重要知识点（一）

Java 8 是目前最常用的 JDK 版本，在增强代码可读性、简化代码方面，相比 Java 7 增加了很多功能，比如 Lambda、Stream 流操作、并行流（ParallelStream）、Optional 可空类型、新日期时间类型等。

这个课程中的所有案例，都充分使用了 Java 8 的各种特性来简化代码。这也就意味着，如果你不了解这些特性的话，理解课程内的 Demo 可能会有些困难。因此，我将这些特性，单独拎了出来组成了两篇加餐。由于后面有单独一节课去讲 Java 8 的日期时间类型，所以这里就不赘述了。

## 如何在项目中用上 Lambda 表达式和 Stream 操作？

Java 8 的特性有很多，除了这两篇加餐外，我再给你推荐一本全面介绍 Java 8 的书，叫《Java 实战（第二版）》。此外，有同学在留言区问，怎么把 Lambda 表达式和 Stream 操作运用到项目中。其实，业务代码中可以使用这些特性的地方有很多。


这里，为了帮助你学习，并把这些特性用到业务开发中，我有三个小建议。

第一，从 List 的操作开始，先尝试把遍历 List 来筛选数据和转换数据的操作，使用 Stream 的 filter 和 map 实现，这是 Stream 最常用、最基本的两个 API。你可以重点看看接下来两节的内容来入门。

第二，使用高级的 IDE 来写代码，以此找到可以利用 Java 8 语言特性简化代码的地方。比如，对于 IDEA，我们可以把匿名类型使用 Lambda 替换的检测规则，设置为 Error 级别严重程度：

6707ccf4415c2d8715ed2529cfdec877.png

这样运行 IDEA 的 Inspect Code 的功能，可以在 Error 级别的错误中看到这个问题，引起更多关注，帮助我们建立使用 Lambda 表达式的习惯：

5062b3ef6ec57ccde0f3f4b182811be4.png

第三，如果你不知道如何把匿名类转换为 Lambda 表达式，可以借助 IDE 来重构：

5a55c4284e4b10f659b7bcf0129cbde7.png

反过来，如果你在学习课程内案例时，如果感觉阅读 Lambda 表达式和 Stream API 比较吃力，同样可以借助 IDE 把 Java 8 的写法转换为使用循环的写法：

98828a36d6bb7b7972a647b37a64f08a.png

或者是把 Lambda 表达式替换为匿名类：

ee9401683b19e57462cb2574c285d67c.png

## Lambda 表达式

Lambda 表达式的初衷是，进一步简化匿名类的语法（不过实现上，Lambda 表达式并不是匿名类的语法糖），使 Java 走向函数式编程。对于匿名类，虽然没有类名，但还是要给出方法定义。这里有个例子，分别使用匿名类和 Lambda 表达式创建一个线程打印字符串：


```

//匿名类
new Thread(new Runnable(){
    @Override
    public void run(){
        System.out.println("hello1");
    }
}).start();
//Lambda表达式
new Thread(() -> System.out.println("hello2")).start();
```

那么，Lambda 表达式如何匹配 Java 的类型系统呢？

答案就是，函数式接口。

函数式接口是一种只有单一抽象方法的接口，使用 @FunctionalInterface 来描述，可以隐式地转换成 Lambda 表达式。使用 Lambda 表达式来实现函数式接口，不需要提供类名和方法定义，通过一行代码提供函数式接口的实例，就可以让函数成为程序中的头等公民，可以像普通数据一样作为参数传递，而不是作为一个固定的类中的固定方法。

那，函数式接口到底是什么样的呢？java.util.function 包中定义了各种函数式接口。比如，用于提供数据的 Supplier 接口，就只有一个 get 抽象方法，没有任何入参、有一个返回值：



```

@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```
我们可以使用 Lambda 表达式或方法引用，来得到 Supplier 接口的实例：


```

//使用Lambda表达式提供Supplier接口实现，返回OK字符串
Supplier<String> stringSupplier = ()->"OK";
//使用方法引用提供Supplier接口实现，返回空字符串
Supplier<String> supplier = String::new;
```
这样，是不是很方便？为了帮你掌握函数式接口及其用法，我再举几个使用 Lambda 表达式或方法引用来构建函数的例子：


```

//Predicate接口是输入一个参数，返回布尔值。我们通过and方法组合两个Predicate条件，判断是否值大于0并且是偶数
Predicate<Integer> positiveNumber = i -> i > 0;
Predicate<Integer> evenNumber = i -> i % 2 == 0;
assertTrue(positiveNumber.and(evenNumber).test(2));

//Consumer接口是消费一个数据。我们通过andThen方法组合调用两个Consumer，输出两行abcdefg
Consumer<String> println = System.out::println;
println.andThen(println).accept("abcdefg");

//Function接口是输入一个数据，计算后输出一个数据。我们先把字符串转换为大写，然后通过andThen组合另一个Function实现字符串拼接
Function<String, String> upperCase = String::toUpperCase;
Function<String, String> duplicate = s -> s.concat(s);
assertThat(upperCase.andThen(duplicate).apply("test"), is("TESTTEST"));

//Supplier是提供一个数据的接口。这里我们实现获取一个随机数
Supplier<Integer> random = ()->ThreadLocalRandom.current().nextInt();
System.out.println(random.get());

//BinaryOperator是输入两个同类型参数，输出一个同类型参数的接口。这里我们通过方法引用获得一个整数加法操作，通过Lambda表达式定义一个减法操作，然后依次调用
BinaryOperator<Integer> add = Integer::sum;
BinaryOperator<Integer> subtraction = (a, b) -> a - b;
assertThat(subtraction.apply(add.apply(1, 2), 3), is(0));
```
Predicate、Function 等函数式接口，还使用 default 关键字实现了几个默认方法。这样一来，它们既可以满足函数式接口只有一个抽象方法，又能为接口提供额外的功能：


```

@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
}
```

很明显，Lambda 表达式给了我们复用代码的更多可能性：我们可以把一大段逻辑中变化的部分抽象出函数式接口，由外部方法提供函数实现，重用方法内的整体逻辑处理。

不过需要注意的是，在自定义函数式接口之前，可以先确认下java.util.function 包中的 43 个标准函数式接口是否能满足需求，我们要尽可能重用这些接口，因为使用大家熟悉的标准接口可以提高代码的可读性。
`https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html`

## 使用 Java 8 简化代码

这一部分，我会通过几个具体的例子，带你感受一下使用 Java 8 简化代码的三个重要方面：
* 使用 Stream 简化集合操作；
* 使用 Optional 简化判空逻辑；
* JDK8 结合 Lambda 和 Stream 对各种类的增强。

## 使用 Stream 简化集合操作

Lambda 表达式可以帮我们用简短的代码实现方法的定义，给了我们复用代码的更多可能性。利用这个特性，我们可以把集合的投影、转换、过滤等操作抽象成通用的接口，然后通过 Lambda 表达式传入其具体实现，这也就是 Stream 操作。

我们看一个具体的例子。这里有一段 20 行左右的代码，实现了如下的逻辑：

* 把整数列表转换为 Point2D 列表；
* 遍历 Point2D 列表过滤出 Y 轴 >1 的对象；
* 计算 Point2D 点到原点的距离；
* 累加所有计算出的距离，并计算距离的平均值。


```

private static double calc(List<Integer> ints) {
    //临时中间集合
    List<Point2D> point2DList = new ArrayList<>();
    for (Integer i : ints) {
        point2DList.add(new Point2D.Double((double) i % 3, (double) i / 3));
    }
    //临时变量，纯粹是为了获得最后结果需要的中间变量
    double total = 0;
    int count = 0;

    for (Point2D point2D : point2DList) {
        //过滤
        if (point2D.getY() > 1) {
            //算距离
            double distance = point2D.distance(0, 0);
            total += distance;
            count++;
        }
    }
    //注意count可能为0的可能
    return count >0 ? total / count : 0;
}
```

现在，我们可以使用 Stream 配合 Lambda 表达式来简化这段代码。简化后一行代码就可以实现这样的逻辑，更重要的是代码可读性更强了，通过方法名就可以知晓大概是在做什么事情。比如：

* map 方法传入的是一个 Function，可以实现对象转换；
* filter 方法传入一个 Predicate，实现对象的布尔判断，只保留返回 true 的数据；
* mapToDouble 用于把对象转换为 double；
* 通过 average 方法返回一个 OptionalDouble，代表可能包含值也可能不包含值的可空 double。

下面的第三行代码，就实现了上面方法的所有工作：


```

List<Integer> ints = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
double average = calc(ints);
double streamResult = ints.stream()
        .map(i -> new Point2D.Double((double) i % 3, (double) i / 3))
        .filter(point -> point.getY() > 1)
        .mapToDouble(point -> point.distance(0, 0))
        .average()
        .orElse(0);
//如何用一行代码来实现，比较一下可读性
assertThat(average, is(streamResult));
```
到这里，你可能会问了，OptionalDouble 又是怎么回事儿？

## 有关 Optional 可空类型

其实，类似 OptionalDouble、OptionalInt、OptionalLong 等，是服务于基本类型的可空对象。此外，Java8 还定义了用于引用类型的 Optional 类。使用 Optional，不仅可以避免使用 Stream 进行级联调用的空指针问题；更重要的是，它提供了一些实用的方法帮我们避免判空逻辑。

如下是一些例子，演示了如何使用 Optional 来避免空指针，以及如何使用它的 fluent API 简化冗长的 if-else 判空逻辑：



```

@Test(expected = IllegalArgumentException.class)
public void optional() {
    //通过get方法获取Optional中的实际值
    assertThat(Optional.of(1).get(), is(1));
    //通过ofNullable来初始化一个null，通过orElse方法实现Optional中无数据的时候返回一个默认值
    assertThat(Optional.ofNullable(null).orElse("A"), is("A"));
    //OptionalDouble是基本类型double的Optional对象，isPresent判断有无数据
    assertFalse(OptionalDouble.empty().isPresent());
    //通过map方法可以对Optional对象进行级联转换，不会出现空指针，转换后还是一个Optional
    assertThat(Optional.of(1).map(Math::incrementExact).get(), is(2));
    //通过filter实现Optional中数据的过滤，得到一个Optional，然后级联使用orElse提供默认值
    assertThat(Optional.of(1).filter(integer -> integer % 2 == 0).orElse(null), is(nullValue()));
    //通过orElseThrow实现无数据时抛出异常
    Optional.empty().orElseThrow(IllegalArgumentException::new);
}
```

c8a901bb16b9fca07ae0fc8bb222b252.jpg

## Java 8 类对于函数式 API 的增强

除了 Stream 之外，Java 8 中有很多类也都实现了函数式的功能。

比如，要通过 HashMap 实现一个缓存的操作，在 Java 8 之前我们可能会写出这样的 getProductAndCache 方法：先判断缓存中是否有值；如果没有值，就从数据库搜索取值；最后，把数据加入缓存。



```

private Map<Long, Product> cache = new ConcurrentHashMap<>();

private Product getProductAndCache(Long id) {
    Product product = null;
    //Key存在，返回Value
    if (cache.containsKey(id)) {
        product = cache.get(id);
    } else {
        //不存在，则获取Value
        //需要遍历数据源查询获得Product
        for (Product p : Product.getData()) {
            if (p.getId().equals(id)) {
                product = p;
                break;
            }
        }
        //加入ConcurrentHashMap
        if (product != null)
            cache.put(id, product);
    }
    return product;
}

@Test
public void notcoolCache() {
    getProductAndCache(1L);
    getProductAndCache(100L);

    System.out.println(cache);
    assertThat(cache.size(), is(1));
    assertTrue(cache.containsKey(1L));
}
```

而在 Java 8 中，我们利用 ConcurrentHashMap 的 computeIfAbsent 方法，用一行代码就可以实现这样的繁琐操作：



```

private Product getProductAndCacheCool(Long id) {
    return cache.computeIfAbsent(id, i -> //当Key不存在的时候提供一个Function来代表根据Key获取Value的过程
            Product.getData().stream()
                    .filter(p -> p.getId().equals(i)) //过滤
                    .findFirst() //找第一个，得到Optional<Product>
                    .orElse(null)); //如果找不到Product，则使用null
}

@Test
public void coolCache()
{
    getProductAndCacheCool(1L);
    getProductAndCacheCool(100L);

    System.out.println(cache);
    assertThat(cache.size(), is(1));
    assertTrue(cache.containsKey(1L));
}
```

computeIfAbsent 方法在逻辑上相当于：



```

if (map.get(key) == null) {
  V newValue = mappingFunction.apply(key);
  if (newValue != null)
    map.put(key, newValue);
}
```
又比如，利用 Files.walk 返回一个 Path 的流，通过两行代码就能实现递归搜索 +grep 的操作。整个逻辑是：递归搜索文件夹，查找所有的.java 文件；然后读取文件每一行内容，用正则表达式匹配 public class 关键字；最后输出文件名和这行内容。


```

@Test
public void filesExample() throws IOException {
    //无限深度，递归遍历文件夹
    try (Stream<Path> pathStream = Files.walk(Paths.get("."))) {
        pathStream.filter(Files::isRegularFile) //只查普通文件
                .filter(FileSystems.getDefault().getPathMatcher("glob:**/*.java")::matches) //搜索java源码文件
                .flatMap(ThrowingFunction.unchecked(path ->
                        Files.readAllLines(path).stream() //读取文件内容，转换为Stream<List>
                        .filter(line -> Pattern.compile("public class").matcher(line).find()) //使用正则过滤带有public class的行
                        .map(line -> path.getFileName() + " >> " + line))) //把这行文件内容转换为文件名+行
                .forEach(System.out::println); //打印所有的行
    }
}
```
输出结果如下：

84349a90ef4aaf30032d0a8f64ab4512.png

我再和你分享一个小技巧吧。因为 Files.readAllLines 方法会抛出一个受检异常（IOException），所以我使用了一个自定义的函数式接口，用 ThrowingFunction 包装这个方法，把受检异常转换为运行时异常，让代码更清晰：



```

@FunctionalInterface
public interface ThrowingFunction<T, R, E extends Throwable> {
    static <T, R, E extends Throwable> Function<T, R> unchecked(ThrowingFunction<T, R, E> f) {
        return t -> {
            try {
                return f.apply(t);
            } catch (Throwable e) {
                throw new RuntimeException(e);
            }
        };
    }

    R apply(T t) throws E;
}
```

如果用 Java 7 实现类似逻辑的话，大概需要几十行代码，你可以尝试下。
## 并行流


前面我们看到的 Stream 操作都是串行 Stream，操作只是在一个线程中执行，此外 Java 8 还提供了并行流的功能：通过 parallel 方法，一键把 Stream 转换为并行操作提交到线程池处理。

比如，如下代码通过线程池来并行消费处理 1 到 100：


```

IntStream.rangeClosed(1,100).parallel().forEach(i->{
    System.out.println(LocalDateTime.now() + " : " + i);
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) { }
});
```
并行流不确保执行顺序，并且因为每次处理耗时 1 秒，所以可以看到在 8 核机器上，数组是按照 8 个一组 1 秒输出一次：



# 32 | 加餐2：带你吃透课程中Java 8的那些重要知识点（二）

上一讲的几个例子中，其实都涉及了 Stream API 的最基本使用方法。今天，我会与你详细介绍复杂、功能强大的 Stream API。

Stream 流式操作，用于对集合进行投影、转换、过滤、排序等，更进一步地，这些操作能链式串联在一起使用，类似于 SQL 语句，可以大大简化代码。可以说，Stream 操作是 Java 8 中最重要的内容，也是这个课程大部分代码都会用到的操作。

我先说明下，有些案例可能不太好理解，建议你对着代码逐一到源码中查看 Stream 操作的方法定义，以及 JDK 中的代码注释。

## Stream 操作详解

为了方便你理解 Stream 的各种操作，以及后面的案例，我先把这节课涉及的 Stream 操作汇总到了一张图中。你可以先熟悉一下。

44a6f4cb8b413ef62c40a272cb474104.jpg

在接下来的讲述中，我会围绕订单场景，给出如何使用 Stream 的各种 API 完成订单的统计、搜索、查询等功能，和你一起学习 Stream 流式操作的各种方法。你可以结合代码中的注释理解案例，也可以自己运行源码观察输出。

我们先定义一个订单类、一个订单商品类和一个顾客类，用作后续 Demo 代码的数据结构：



```

//订单类
@Data
public class Order {
    private Long id;
    private Long customerId;//顾客ID
    private String customerName;//顾客姓名
    private List<OrderItem> orderItemList;//订单商品明细
    private Double totalPrice;//总价格
    private LocalDateTime placedAt;//下单时间
}
//订单商品类
@Data
@AllArgsConstructor
@NoArgsConstructor
public class OrderItem {
    private Long productId;//商品ID
    private String productName;//商品名称
    private Double productPrice;//商品价格
    private Integer productQuantity;//商品数量
}
//顾客类
@Data
@AllArgsConstructor
public class Customer {
    private Long id;
    private String name;//顾客姓名
}
```

在这里，我们有一个 orders 字段保存了一些模拟数据，类型是 List。这里，我就不贴出生成模拟数据的代码了。这不会影响你理解后面的代码，你也可以自己下载源码阅读。

## 创建流
要使用流，就要先创建流。创建流一般有五种方式：
* 通过 stream 方法把 List 或数组转换为流；
* 通过 Stream.of 方法直接传入多个元素构成一个流；
* 通过 Stream.iterate 方法使用迭代的方式构造一个无限流，然后使用 limit 限制流元素个数；
* 通过 Stream.generate 方法从外部传入一个提供元素的 Supplier 来构造无限流，然后使用 limit 限制流元素个数；
* 通过 IntStream 或 DoubleStream 构造基本类型的流。


```

//通过stream方法把List或数组转换为流
@Test
public void stream()
{
    Arrays.asList("a1", "a2", "a3").stream().forEach(System.out::println);
    Arrays.stream(new int[]{1, 2, 3}).forEach(System.out::println);
}

//通过Stream.of方法直接传入多个元素构成一个流
@Test
public void of()
{
    String[] arr = {"a", "b", "c"};
    Stream.of(arr).forEach(System.out::println);
    Stream.of("a", "b", "c").forEach(System.out::println);
    Stream.of(1, 2, "a").map(item -> item.getClass().getName()).forEach(System.out::println);
}

//通过Stream.iterate方法使用迭代的方式构造一个无限流，然后使用limit限制流元素个数
@Test
public void iterate()
{
    Stream.iterate(2, item -> item * 2).limit(10).forEach(System.out::println);
    Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.TEN)).limit(10).forEach(System.out::println);
}

//通过Stream.generate方法从外部传入一个提供元素的Supplier来构造无限流，然后使用limit限制流元素个数
@Test
public void generate()
{
    Stream.generate(() -> "test").limit(3).forEach(System.out::println);
    Stream.generate(Math::random).limit(10).forEach(System.out::println);
}

//通过IntStream或DoubleStream构造基本类型的流
@Test
public void primitive()
{
    //演示IntStream和DoubleStream
    IntStream.range(1, 3).forEach(System.out::println);
    IntStream.range(0, 3).mapToObj(i -> "x").forEach(System.out::println);

    IntStream.rangeClosed(1, 3).forEach(System.out::println);
    DoubleStream.of(1.1, 2.2, 3.3).forEach(System.out::println);

    //各种转换，后面注释代表了输出结果
    System.out.println(IntStream.of(1, 2).toArray().getClass()); //class [I
    System.out.println(Stream.of(1, 2).mapToInt(Integer::intValue).toArray().getClass()); //class [I
    System.out.println(IntStream.of(1, 2).boxed().toArray().getClass()); //class [Ljava.lang.Object;
    System.out.println(IntStream.of(1, 2).asDoubleStream().toArray().getClass()); //class [D
    System.out.println(IntStream.of(1, 2).asLongStream().toArray().getClass()); //class [J

    //注意基本类型流和装箱后的流的区别
    Arrays.asList("a", "b", "c").stream()   // Stream<String>
            .mapToInt(String::length)       // IntStream
            .asLongStream()                 // LongStream
            .mapToDouble(x -> x / 10.0)     // DoubleStream
            .boxed()                        // Stream<Double>
            .mapToLong(x -> 1L)             // LongStream
            .mapToObj(x -> "")              // Stream<String>
            .collect(Collectors.toList());
}
```

## filter

filter 方法可以实现过滤操作，类似 SQL 中的 where。我们可以使用一行代码，通过 filter 方法实现查询所有订单中最近半年金额大于 40 的订单，通过连续叠加 filter 方法进行多次条件过滤：



```

//最近半年的金额大于40的订单
orders.stream()
        .filter(Objects::nonNull) //过滤null值
        .filter(order -> order.getPlacedAt().isAfter(LocalDateTime.now().minusMonths(6))) //最近半年的订单
        .filter(order -> order.getTotalPrice() > 40) //金额大于40的订单
        .forEach(System.out::println);  
```
如果不使用 Stream 的话，必然需要一个中间集合来收集过滤后的结果，而且所有的过滤条件会堆积在一起，代码冗长且不易读。

## map

map 操作可以做转换（或者说投影），类似 SQL 中的 select。为了对比，我用两种方式统计订单中所有商品的数量，前一种是通过两次遍历实现，后一种是通过两次 mapToLong+sum 方法实现：



```

//计算所有订单商品数量
//通过两次遍历实现
LongAdder longAdder = new LongAdder();
orders.stream().forEach(order ->
        order.getOrderItemList().forEach(orderItem -> longAdder.add(orderItem.getProductQuantity())));

//使用两次mapToLong+sum方法实现
assertThat(longAdder.longValue(), is(orders.stream().mapToLong(order ->
        order.getOrderItemList().stream()
                .mapToLong(OrderItem::getProductQuantity).sum()).sum()));
```
显然，后一种方式无需中间变量 longAdder，更直观。

这里再补充一下，使用 for 循环生成数据，是我们平时常用的操作，也是这个课程会大量用到的。现在，我们可以用一行代码使用 IntStream 配合 mapToObj 替代 for 循环来生成数据，比如生成 10 个 Product 元素构成 List：

```
//把IntStream通过转换Stream<Project>
System.out.println(IntStream.rangeClosed(1,10)
        .mapToObj(i->new Product((long)i, "product"+i, i*100.0))
        .collect(toList()));
```

## flatMap

接下来，我们看看 flatMap 展开或者叫扁平化操作，相当于 map+flat，通过 map 把每一个元素替换为一个流，然后展开这个流。

比如，我们要统计所有订单的总价格，可以有两种方式：

* 直接通过原始商品列表的商品个数 * 商品单价统计的话，可以先把订单通过 flatMap 展开成商品清单，也就是把 Order 替换为 Stream，然后对每一个 OrderItem 用 mapToDouble 转换获得商品总价，最后进行一次 sum 求和；

* 利用 flatMapToDouble 方法把列表中每一项展开替换为一个 DoubleStream，也就是直接把每一个订单转换为每一个商品的总价，然后求和。



```

//直接展开订单商品进行价格统计
System.out.println(orders.stream()
        .flatMap(order -> order.getOrderItemList().stream())
        .mapToDouble(item -> item.getProductQuantity() * item.getProductPrice()).sum());

//另一种方式flatMap+mapToDouble=flatMapToDouble
System.out.println(orders.stream()
        .flatMapToDouble(order ->
                order.getOrderItemList()
                        .stream().mapToDouble(item -> item.getProductQuantity() * item.getProductPrice()))
        .sum());
```
这两种方式可以得到相同的结果，并无本质区别。

## sorted

sorted 操作可以用于行内排序的场景，类似 SQL 中的 order by。比如，要实现大于 50 元订单的按价格倒序取前 5，可以通过 Order::getTotalPrice 方法引用直接指定需要排序的依据字段，通过 reversed() 实现倒序：

```
//大于50的订单,按照订单价格倒序前5
orders.stream().filter(order -> order.getTotalPrice() > 50)
        .sorted(comparing(Order::getTotalPrice).reversed())
        .limit(5)
        .forEach(System.out::println);  
```

## distinct

distinct 操作的作用是去重，类似 SQL 中的 distinct。比如下面的代码实现：

* 查询去重后的下单用户。使用 map 从订单提取出购买用户，然后使用 distinct 去重。

* 查询购买过的商品名。使用 flatMap+map 提取出订单中所有的商品名，然后使用 distinct 去重。



```
//去重的下单用户
System.out.println(orders.stream().map(order -> order.getCustomerName()).distinct().collect(joining(",")));

//所有购买过的商品
System.out.println(orders.stream()
        .flatMap(order -> order.getOrderItemList().stream())
        .map(OrderItem::getProductName)
        .distinct().collect(joining(",")));
```

## skip & limit

skip 和 limit 操作用于分页，类似 MySQL 中的 limit。其中，skip 实现跳过一定的项，limit 用于限制项总数。比如下面的两段代码：

* 按照下单时间排序，查询前 2 个订单的顾客姓名和下单时间；
* 按照下单时间排序，查询第 3 和第 4 个订单的顾客姓名和下单时间。

```
//按照下单时间排序，查询前2个订单的顾客姓名和下单时间
orders.stream()
        .sorted(comparing(Order::getPlacedAt))
        .map(order -> order.getCustomerName() + "@" + order.getPlacedAt())
        .limit(2).forEach(System.out::println);
//按照下单时间排序，查询第3和第4个订单的顾客姓名和下单时间
orders.stream()
        .sorted(comparing(Order::getPlacedAt))
        .map(order -> order.getCustomerName() + "@" + order.getPlacedAt())
        .skip(2).limit(2).forEach(System.out::println);
```

## collect

collect 是收集操作，对流进行终结（终止）操作，把流导出为我们需要的数据结构。“终结”是指，导出后，无法再串联使用其他中间操作，比如 filter、map、flatmap、sorted、distinct、limit、skip。
 
在 Stream 操作中，collect 是最复杂的终结操作，比较简单的终结操作还有 forEach、toArray、min、max、count、anyMatch 等，我就不再展开了，你可以查询JDK 文档，搜索 terminal operation 或 intermediate operation。

查询JDK 文档: `https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html`

接下来，我通过 6 个案例，来演示下几种比较常用的 collect 操作：
* 第一个案例，实现了字符串拼接操作，生成一定位数的随机字符串。
* 第二个案例，通过 Collectors.toSet 静态方法收集为 Set 去重，得到去重后的下单用户，再通过 Collectors.joining 静态方法实现字符串拼接。
* 第三个案例，通过 Collectors.toCollection 静态方法获得指定类型的集合，比如把 List转换为 LinkedList。
* 第四个案例，通过 Collectors.toMap 静态方法将对象快速转换为 Map，Key 是订单 ID、Value 是下单用户名。
* 第五个案例，通过 Collectors.toMap 静态方法将对象转换为 Map。Key 是下单用户名，Value 是下单时间，一个用户可能多次下单，所以直接在这里进行了合并，只获取最近一次的下单时间。
* 第六个案例，使用 Collectors.summingInt 方法对商品数量求和，再使用 Collectors.averagingInt 方法对结果求平均值，以统计所有订单平均购买的商品数量。

```

//生成一定位数的随机字符串
System.out.println(random.ints(48, 122)
    .filter(i -> (i < 57 || i > 65) && (i < 90 || i > 97))
    .mapToObj(i -> (char) i)
    .limit(20)
    .collect(StringBuilder::new, StringBuilder::append, StringBuilder::append)
    .toString());

//所有下单的用户，使用toSet去重后实现字符串拼接
System.out.println(orders.stream()
    .map(order -> order.getCustomerName()).collect(toSet())
    .stream().collect(joining(",", "[", "]")));

//用toCollection收集器指定集合类型
System.out.println(orders.stream().limit(2).collect(toCollection(LinkedList::new)).getClass());

//使用toMap获取订单ID+下单用户名的Map
orders.stream()
    .collect(toMap(Order::getId, Order::getCustomerName))
    .entrySet().forEach(System.out::println);

//使用toMap获取下单用户名+最近一次下单时间的Map
orders.stream()
    .collect(toMap(Order::getCustomerName, Order::getPlacedAt, (x, y) -> x.isAfter(y) ? x : y))
    .entrySet().forEach(System.out::println);

//订单平均购买的商品数量
System.out.println(orders.stream().collect(averagingInt(order ->
    order.getOrderItemList().stream()
            .collect(summingInt(OrderItem::getProductQuantity)))));
            
```

可以看到，这 6 个操作使用 Stream 方式一行代码就可以实现，但使用非 Stream 方式实现的话，都需要几行甚至十几行代码。

有关 Collectors 类的一些常用静态方法，我总结到了一张图中，你可以再整理一下思路：

![](/static/image/5af5ba60d7af2c8780b69bc6c71cf3de.png)

其中，groupBy 和 partitionBy 比较复杂，我和你举例介绍。



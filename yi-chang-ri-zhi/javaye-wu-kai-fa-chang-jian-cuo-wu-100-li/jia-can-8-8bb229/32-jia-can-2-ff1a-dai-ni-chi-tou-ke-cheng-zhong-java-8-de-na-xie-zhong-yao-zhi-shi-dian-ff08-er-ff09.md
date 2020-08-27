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












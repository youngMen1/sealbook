# 1.Java8新特性之日期处理

## 1.1.简介

伴随`lambda表达式`、`streams`以及一系列小优化，Java 8 推出了全新的日期时间API。

Java处理日期、日历和时间的不足之处：将 java.util.Date 设定为可变类型，以及 SimpleDateFormat 的非线程安全使其应用非常受限。然后就在 java8 上面增加新的特性。

全新API的众多好处之一就是，明确了日期时间概念，例如：`瞬时（instant）`、`长短（duration）`、`日期`、`时间`、`时区`和`周期`。

同时继承了Joda 库按人类语言和计算机各自解析的时间处理方式。不同于老版本，新API基于ISO标准日历系统，**java.time包下的所有类都是不可变类型而且线程安全**。

## 1.2.关键类 {#item-2}

* Instant：瞬时实例。
* LocalDate：本地日期，不包含具体时间 例如：2014-01-14 可以用来记录生日、纪念日、加盟日等。
* LocalTime：本地时间，不包含日期。
* LocalDateTime：组合了日期和时间，但不包含时差和时区信息。
* ZonedDateTime：最完整的日期时间，包含时区和相对UTC或格林威治的时差。

新API还引入了 ZoneOffSet 和 ZoneId 类，使得解决时区问题更为简便。解析、格式化时间的**DateTimeFormatter类**也全部重新设计。

# 2.实战

在教程中我们将通过一些简单的实例来学习如何使用新API，因为只有在实际的项目中用到，才是学习新知识以及新技术最快的方式。

### 1. 获取当前的日期 {#item-3-1}

Java 8 中的`LocalDate`用于表示当天日期。和 java.util.Date不同，它只有日期，不包含时间。当你仅需要表示日期时就用这个类。

```
// 获取今天的日期
public void getCurrentDate(){
    LocalDate today = LocalDate.now();
    System.out.println("Today's Local date : " + today);

    // 这个是作为对比
    Date date = new Date();
    System.out.println(date);
    
}
```

```
获取今天的日期:2020-06-16
Tue Jun 16 09:34:35 CST 2020
```

上面的代码创建了当天的日期，不含时间信息。打印出的日期格式非常友好，不像 Date类 打印出一堆没有格式化的信息。

### 2. 获取年、月、日信息 {#item-3-2}

### 3.处理特定日期 {#item-3-3}

### 4.判断两个日期是否相等 {#item-3-4}

### 5.检查像生日这种周期性事件 {#item-3-5}

### 6.获取当前时间 {#item-3-6}

### 7.在现有的时间上增加小时 {#item-3-7}

### 8.如何计算一个星期之后的日期 {#item-3-8}

### 9.计算一年前或一年后的日期 {#item-3-9}

### 10.使用Java 8的Clock时钟类 {#item-3-10}

### 11.判断日期是早于还是晚于另一个日期 {#item-3-11}

### 12.处理时区 {#item-3-12}

### 13.如何体现出固定日期 {#item-3-13}

### 14.检查闰年 {#item-3-14}

### 15.计算两个日期之间的天数和月数 {#item-3-15}

### 16.包含时差信息的日期和时间 {#item-3-16}

### 17.获取当前的时间戳 {#item-3-17}

### 18.使用预定义的格式化工具去解析或格式化日期 {#item-3-18}

# 3.总结

1.提供了javax.time.ZoneId 获取时区。

2.提供了LocalDate和LocalTime类。

3.Java 8 的所有日期和时间API都是不可变类并且线程安全，而现有的Date和Calendar API中的java.util.Date和SimpleDateFormat是非线程安全的。

4.主包是 java.time,包含了表示日期、时间、时间间隔的一些类。里面有两个子包java.time.format用于格式化， java.time.temporal用于更底层的操作。

5.时区代表了地球上某个区域内普遍使用的标准时间。每个时区都有一个代号，格式通常由区域/城市构成（Asia/Tokyo），在加上与格林威治或 UTC的时差。例如：东京的时差是+09:00。

# 4.参考

Java8新特性之日期处理：  
[https://segmentfault.com/a/1190000012922933](https://segmentfault.com/a/1190000012922933)


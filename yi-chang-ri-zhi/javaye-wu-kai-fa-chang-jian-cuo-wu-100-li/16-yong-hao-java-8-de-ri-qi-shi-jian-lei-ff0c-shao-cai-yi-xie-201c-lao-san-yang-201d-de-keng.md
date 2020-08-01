# 1.用好Java 8的日期时间类，少踩一些“老三样”的坑
在 Java 8 之前，我们处理日期时间需求时，使用 Date、Calender 和 SimpleDateFormat，来声明时间戳、使用日历处理日期和格式化解析日期时间。但是，这些类的 API 的缺点比较明显，比如可读性差、易用性差、使用起来冗余繁琐，还有线程安全问题。

因此，Java 8 推出了新的日期时间类。每一个类功能明确清晰、类之间协作简单、API 定义清晰不踩坑，API 功能强大无需借助外部工具类即可完成操作，并且线程安全。

但是，Java 8 刚推出的时候，诸如序列化、数据访问等类库都还不支持 Java 8 的日期时间类型，需要在新老类中来回转换。比如，在业务逻辑层使用 LocalDateTime，存入数据库或者返回前端的时候还要切换回 Date。因此，很多同学还是选择使用老的日期时间类。

现在几年时间过去了，几乎所有的类库都支持了新日期时间类型，使用起来也不会有来回切换等问题了。但，很多代码中因为还是用的遗留的日期时间类，因此出现了很多时间错乱的错误实践。比如，试图通过随意修改时区，使读取到的数据匹配当前时钟；再比如，试图直接对读取到的数据做加、减几个小时的操作，来“修正数据”。

今天，我就重点与你分析下时间错乱问题背后的原因，看看使用遗留的日期时间类，来处理日期时间初始化、格式化、解析、计算等可能会遇到的问题，以及如何使用新日期时间类来解决。

## 初始化日期时间

我们先从日期时间的初始化看起。如果要初始化一个 2019 年 12 月 31 日 11 点 12 分 13 秒这样的时间，可以使用下面的两行代码吗？

```
Date date = new Date(2019, 12, 31, 11, 12, 13);
System.out.println(date);
```

可以看到，输出的时间是 3029 年 1 月 31 日 11 点 12 分 13 秒：

```
Sat Jan 31 11:12:13 CST 3920
```
相信看到这里，你会说这是新手才会犯的低级错误：年应该是和 1900 的差值，月应该是从 0 到 11 而不是从 1 到 12。

```
Date date = new Date(2019 - 1900, 11, 31, 11, 12, 13);
```

你说的没错，但更重要的问题是，当有国际化需求时，需要使用 Calendar 类来初始化时间。

使用 Calendar 改造之后，初始化时年参数直接使用当前年即可，不过月需要注意是从 0 到 11。当然，你也可以直接使用 Calendar.DECEMBER 来初始化月份，更不容易犯错。为了说明时区的问题，我分别使用当前时区和纽约时区初始化了两次相同的日期：

```
Calendar calendar = Calendar.getInstance();
calendar.set(2019, 11, 31, 11, 12, 13);
System.out.println(calendar.getTime());
Calendar calendar2 = Calendar.getInstance(TimeZone.getTimeZone("America/New_York"));
calendar2.set(2019, Calendar.DECEMBER, 31, 11, 12, 13);
System.out.println(calendar2.getTime());
```
输出显示了两个时间，说明时区产生了作用。但，我们更习惯年 / 月 / 日 时: 分: 秒这样的日期时间格式，对现在输出的日期格式还不满意：

```
Tue Dec 31 11:12:13 CST 2019
Wed Jan 01 00:12:13 CST 2020
```
## “恼人”的时区问题

我们知道，全球有 24 个时区，同一个时刻不同时区（比如中国上海和美国纽约）的时间是不一样的。对于需要全球化的项目，如果初始化时间时没有提供时区，那就不是一个真正意义上的时间，只能认为是我看到的当前时间的一个表示。

关于 Date 类，我们要有两点认识：

* 一是，Date 并无时区问题，世界上任何一台计算机使用 new Date() 初始化得到的时间都一样。因为，Date 中保存的是 UTC 时间，UTC 是以原子钟为基础的统一时间，不以太阳参照计时，并无时区划分。
* 二是，Date 中保存的是一个时间戳，代表的是从 1970 年 1 月 1 日 0 点（Epoch 时间）到现在的毫秒数。尝试输出 Date(0)：

```
System.out.println(new Date(0));
System.out.println(TimeZone.getDefault().getID() + ":" + TimeZone.getDefault().getRawOffset()/3600000);
```
我得到的是 1970 年 1 月 1 日 8 点。因为我机器当前的时区是中国上海，相比 UTC 时差 +8 小时：

```
Thu Jan 01 08:00:00 CST 1970
Asia/Shanghai:8
```
对于国际化（世界各国的人都在使用）的项目，处理好时间和时区问题首先就是要正确保存日期时间。这里有两种保存方式：
* 方式一，以 UTC 保存，保存的时间没有时区属性，是不涉及时区时间差问题的世界统一时间。我们通常说的时间戳，或 Java 中的 Date 类就是用的这种方式，这也是推荐的方式。

* 方式二，以字面量保存，比如年 / 月 / 日 时: 分: 秒，一定要同时保存时区信息。只有有了时区信息，我们才能知道这个字面量时间真正的时间点，否则它只是一个给人看的时间表示，只在当前时区有意义。Calendar 是有时区概念的，所以我们通过不同的时区初始化 Calendar，得到了不同的时间。

正确保存日期时间之后，就是正确展示，即我们要使用正确的时区，把时间点展示为符合当前时区的时间表示。到这里，我们就能理解为什么会有所谓的“时间错乱”问题了。接下来，我再通过实际案例分析一下，从字面量解析成时间和从时间格式化为字面量这两类问题。

**第一类是**，对于同一个时间表示，比如 2020-01-02 22:00:00，不同时区的人转换成 Date 会得到不同的时间（时间戳）：

```
String stringDate = "2020-01-02 22:00:00";
SimpleDateFormat inputFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//默认时区解析时间表示
Date date1 = inputFormat.parse(stringDate);
System.out.println(date1 + ":" + date1.getTime());
//纽约时区解析时间表示
inputFormat.setTimeZone(TimeZone.getTimeZone("America/New_York"));
Date date2 = inputFormat.parse(stringDate);
System.out.println(date2 + ":" + date2.getTime());
```

可以看到，把 2020-01-02 22:00:00 这样的时间表示，对于当前的上海时区和纽约时区，转化为 UTC 时间戳是不同的时间：


```

Thu Jan 02 22:00:00 CST 2020:1577973600000
Fri Jan 03 11:00:00 CST 2020:1578020400000
```
这正是 UTC 的意义，并不是时间错乱。对于同一个本地时间的表示，不同时区的人解析得到的 UTC 时间一定是不同的，反过来不同的本地时间可能对应同一个 UTC。

**第二类问题是**，格式化后出现的错乱，即同一个 Date，在不同的时区下格式化得到不同的时间表示。比如，在我的当前时区和纽约时区格式化 2020-01-02 22:00:00：



```

String stringDate = "2020-01-02 22:00:00";
SimpleDateFormat inputFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//同一Date
Date date = inputFormat.parse(stringDate);
//默认时区格式化输出：
System.out.println(new SimpleDateFormat("[yyyy-MM-dd HH:mm:ss Z]").format(date));
//纽约时区格式化输出
TimeZone.setDefault(TimeZone.getTimeZone("America/New_York"));
System.out.println(new SimpleDateFormat("[yyyy-MM-dd HH:mm:ss Z]").format(date));
```
输出如下，我当前时区的 Offset（时差）是 +8 小时，对于 -5 小时的纽约，晚上 10 点对应早上 9 点：



```

[2020-01-02 22:00:00 +0800]
[2020-01-02 09:00:00 -0500]
```
因此，有些时候数据库中相同的时间，由于服务器的时区设置不同，读取到的时间表示不同。这，不是时间错乱，正是时区发挥了作用，因为 UTC 时间需要根据当前时区解析为正确的本地时间。

所以，**要正确处理时区，在于存进去和读出来两方面：**存的时候，需要使用正确的当前时区来保存，这样 UTC 时间才会正确；读的时候，也只有正确设置本地时区，才能把 UTC 时间转换为正确的当地时间。

Java 8 推出了新的时间日期类 ZoneId、ZoneOffset、LocalDateTime、ZonedDateTime 和 DateTimeFormatter，处理时区问题更简单清晰。我们再用这些类配合一个完整的例子，来理解一下时间的解析和展示：


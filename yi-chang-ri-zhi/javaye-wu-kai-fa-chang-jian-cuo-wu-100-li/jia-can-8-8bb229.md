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



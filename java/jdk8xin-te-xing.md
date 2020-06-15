# 1.Java8新特性之日期处理

## 1.1.简介

伴随`lambda表达式`、`streams`以及一系列小优化，Java 8 推出了全新的日期时间API。

Java处理日期、日历和时间的不足之处：将 java.util.Date 设定为可变类型，以及 SimpleDateFormat 的非线程安全使其应用非常受限。然后就在 java8 上面增加新的特性。

全新API的众多好处之一就是，明确了日期时间概念，例如：`瞬时（instant）`、`长短（duration）`、`日期`、`时间`、`时区`和`周期`。

同时继承了Joda 库按人类语言和计算机各自解析的时间处理方式。不同于老版本，新API基于ISO标准日历系统，java.time包下的所有类都是不可变类型而且线程安全。

## 1.2.关键类 {#item-2}



# 2.参考

Java8新特性之日期处理：  
[https://segmentfault.com/a/1190000012922933](https://segmentfault.com/a/1190000012922933)


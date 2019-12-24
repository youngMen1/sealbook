# @JsonFormat、@JSONField、@DateTimeFormat的使用和区别

### 三者出处

1、JsonFormat来源于jackson，Jackson是一个简单基于Java应用库，Jackson可以轻松的将Java对象转换成json对象和xml文档，同样也可以将json、xml转换成Java对象。Jackson所依赖的jar包较少，简单易用并且性能也要相对高些，并且Jackson社区相对比较活跃，更新速度也比较快。

2、JSONField来源于fastjson，是阿里巴巴的开源框架，主要进行JSON解析和序列化。

3、DateTimeFormat是spring自带的处理框架，主要用于将时间格式化。

### 作用

@DatetimeFormat是将String转换成Date，一般前台给后台传值时用

@JsonFormat\(pattern="yyyy-MM-dd"\) 将Date转换成String 一般后台传值给前台时



### @DateTimeFormat

因为其用法比较单一，只用于将字符串格式化成日期，在加入spring以后，直接使用注解@DateTimeFormat\(pattern=”yyyy-MM-dd”\)即可。@DateTimeFormat 注解有3个可选的属性：style，pattern和iso。

### @JsonFormat

用法为在属性值上 @JsonFormat\(pattern=”yyyy-MM-dd”,timezone=”GMT+8”\)，如果直接使用 @JsonFormat\(pattern=”yyyy-MM-dd”\)就会出现2018-08-01 08:00:00的情况， 会相差8个小时，因为我们是东八区（北京时间）。所以我们在格式化的时候要指定时区（timezone ）

### @JSONField

用法：目前最长的用属性是@JSONField\(name=”resType”\)和 @JSONField\(format=”yyyy-MM-dd”\)

name：@JSONField\(name=”resType”\)主要用于指定前端传到后台时对应的key值，如果bean中没有这个注解，则默认前端传过来的key是field本身，即如果是private String name，name前端对应的key就是name才能对应上。

format @JSONField\(format=”yyyy-MM-dd”\)主要用于格式化日期，比如前台传过来的时间是2018-07-12 17:44:08，但是通过这个注解，你存到数据库的时间就是2018-07-12 00:00:00.

### 




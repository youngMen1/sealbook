# 日期对比
## @JSONField

@JSONField是fastjson的注解，作用是将日期按照指定的格式，格式化为字符串，返回给前端
**@JSONField的作用对象:**
* 1.Field
@JSONField作用在Field时，其name不仅定义了输入key的名称，同时也定义了输出的名称。

* 2.Setter 和 Getter方法
顾名思义，当作用在setter方法上时，就相当于根据 name 到 json中寻找对应的值，并调用该setter对象赋值。

当作用在getter上时，在bean转换为json时，其key值为name定义的值。

```
@JSONField(format = "yyyy-MM-dd HH:mm:ss")
```

## @JsonFormat

主要用于后台传值到前台

@JsonFormat是Jackson的注解，和@JSONField功能相同，将日期按照指定格式进行格式化，模式的市区是GMT
将Java对象转换成json对象和xml文档，或将json、xml转换成Java对象。

```
@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")

```

## @DateTimeFormat

一般前台给后台传值时用

@DateTimeFormat是Spring的注解，作用是限制前端传入的时间格式，如果格式不匹配，则会抛出异常，可以理解成一种格式限制，不加该注解，Spring也会将前端传入的时间字符串解析成Date类型


```
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
```


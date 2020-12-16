# 日期对比
## @JSONField(format = "yyyy-MM-dd HH:mm:ss")

@JSONField是fastjson的注解，作用是将日期按照指定的格式，格式化为字符串，返回给前端

```
@JSONField(format = "yyyy-MM-dd HH:mm:ss")
```

## @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")

@JsonFormat是Jackson的注解，和@JSONField功能相同，将日期按照指定格式进行格式化，模式的市区是GMT
@DateTimeFormat是Spring的注解，作用是限制前端传入的时间格式，如果格式不匹配，则会抛出异常，可以理解成一种格式限制，不加该注解，Spring也会将前端传入的时间字符串解析成Date类型

```
@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")

```


## @DateTimeFormat

```
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
```


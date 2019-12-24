# SpringBoot注解验证参数

| 注解 | 作用类型 | 解释 |
| :--- | :--- | :--- |
| @NotNull | 任何类型 | 属性不能为null |
| @NotEmpty | 集合 | 集合不能为null，且size大于0 |
| @NotBlank | 字符串、字符 | 字符类不能为null，且去掉空格之后长度大于0 |
| @AssertTrue | Boolean、boolean | 布尔属性必须是true |
| @Min | 数字类型（原子和包装） | 限定数字的最小值（整型） |
| @Max | 同@Min | 限定数字的最大值（整型） |
| @DecimalMin | 同@Min | 限定数字的最小值（字符串，可以是小数） |
| @DecimalMax | 同@Min | 限定数字的最大值（字符串，可以是小数） |
| @Range | 数字类型（原子和包装） | 限定数字范围（长整型） |
| @Length | 字符串 | 限定字符串长度 |
| @Size | 集合 | 限定集合大小 |
| @Past | 时间、日期 | 必须是一个过去的时间或日期 |
| @Future | 时期、时间 | 必须是一个未来的时间或日期 |
| @Email | 字符串 | 必须是一个邮箱格式 |
| @Pattern | 字符串、字符 | 正则匹配字符串 |

### @Pattern的用法:
![img](/static/image/微信截图_20191224104229.png)





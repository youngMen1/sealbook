# 注解使用

@Getter and @Setter

可以用@Getter / @Setter注释字段（也可以注释到类上的—\(在实体类中常用且推荐\)），lombok会自动生成默认的Getter/Setter方法。

@ToString

自动生成toString\(\)方法，默认情况，按顺序（以“,”分隔）打印你的类名称以及每个字段。也可以设置不包含哪些字段/@ToString\(exclude = {“id”,”name”}\)

![](/assets/lombok.png)

@NoArgsConstructor ： 生成一个无参数的构造方法

@AllArgsContructor： ?会生成一个包含所有变量

@RequiredArgsConstructor： 会生成一个包含常量，和标识了NotNull的变量的构造方法。生成的构造方法是私有的private。

主要使用前两个注解，这样就不需要自己写构造方法，代码简洁规范。

### 常用注解 {#常用注解}

@Data ：注解在类上；提供类所有属性的 getting 和 setting 方法，此外还提供了equals、canEqual、hashCode、toString 方法  
@Setter：注解在属性上；为属性提供 setting 方法  
@Getter：注解在属性上；为属性提供 getting 方法  
@Log4j ：注解在类上；为类提供一个 属性名为log 的 log4j 日志对象  
@Slf4j : 注解在类上；为类提供一个 属性名为log 的 log4j 日志对象  
@NoArgsConstructor：注解在类上；为类提供一个无参的构造方法  
@AllArgsConstructor：注解在类上；为类提供一个全参的构造方法  
@Accessors\(chain = true\) 支持链式调用的:  
@Builder 建筑者模式

@NonNull自检是否为空值


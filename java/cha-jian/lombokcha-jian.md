1.注解使用

@Getter and @Setter

可以用@Getter / @Setter注释字段（也可以注释到类上的—\(在实体类中常用且推荐\)），lombok会自动生成默认的Getter/Setter方法。

@ToString

自动生成toString\(\)方法，默认情况，按顺序（以“,”分隔）打印你的类名称以及每个字段。也可以设置不包含哪些字段/@ToString\(exclude = {“id”,”name”}\)

![](/assets/lombok.png)

@NoArgsConstructor ： 生成一个无参数的构造方法

@AllArgsContructor： ?会生成一个包含所有变量

@RequiredArgsConstructor： 会生成一个包含常量，和标识了NotNull的变量的构造方法。生成的构造方法是私有的private。

主要使用前两个注解，这样就不需要自己写构造方法，代码简洁规范。


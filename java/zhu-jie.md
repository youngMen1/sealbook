# 1.SpringBoot注解

## 1.1.**注解\(annotations\)列表**

**@SpringBootApplication**：

包含了@ComponentScan、@Configuration和@EnableAutoConfiguration注解。其中@ComponentScan让spring Boot扫描到Configuration类并把它加入到程序上下文。

**@Configuration: **注解在类上，表示这是一个IOC容器，相当于spring的配置文件，java配置的方式。 IOC容器的配置类一般与 @Bean 注解配合使用，用 @Configuration 注解类等价与 XML 中配置 beans，用@Bean 注解方法等价于 XML 中配置 bean。

**@Bean： **注解在方法上，声明当前方法返回一个Bean

**@Autowired：**按类型注入.默认属性required= true;当不能确定 Spring 容器中一定拥有某个类的Bean 时， 可以在需要自动注入该类 Bean 的地方可以使用 @Autowired\(required = false\)， 这等于告诉Spring：在找不到匹配Bean时也不抛出BeanCreationException 异常。@Autowired 和 @Qualifier 结合使用时，自动注入的策略就从 byType 转变byName 了。@Autowired可以对成员变量、方法以及构造函数进行注释，而 @Qualifier 的标注对象是成员变量、方法入参、构造函数入参。正是由于注释对象的不同，所以 Spring 不将 @Autowired 和 @Qualifier 统一成一个注释类，@Resource： 按名称装配

区别：@Resource默认按照名称方式进行bean匹配，@Autowired默认按照类型方式进行bean匹配

**@Resource:  **默认按照名称方式进行bean匹配

**@Service: **注解在类上，表示这是一个业务层bean

**@Controller：**注解在类上，表示这是一个控制层bean

**@Repository: ** 注解在类上，表示这是一个数据访问层bean

**@Component：** 注解在类上，表示通用bean，value不写默认就是类名首字母小写

**@Scope：**注解在类上，描述spring容器如何创建Bean实例。

（1）singleton： 表示在spring容器中的单例，通过spring容器获得该bean时总是返回唯一的实例

（2）prototype：表示每次获得bean都会生成一个新的对象

（3）request：表示在一次http请求内有效（只适用于web应用）

（4）session：表示在一个用户会话内有效（只适用于web应用）

（5）globalSession：表示在全局会话内有效（只适用于web应用）

**@Value：**注解在变量上，从配置文件中读取。

## 注解优势

1.采用纯java代码，不在需要配置繁杂的xml文件

2.在配置中也可享受面向对象带来的好处

3.类型安全对重构可以提供良好的支持

4.减少复杂配置文件的同时亦能享受到springIoC容器提供的功能

## 参考

[https://blog.csdn.net/yitian\_66/article/details/80866571](https://blog.csdn.net/yitian_66/article/details/80866571)


# 1.SpringBoot注解

## 1.1.**注解\(annotations\)列表**

**@SpringBootApplication**：

包含了@ComponentScan、@Configuration和@EnableAutoConfiguration注解。其中@ComponentScan让spring Boot扫描到Configuration类并把它加入到程序上下文。

**@EnableAutoConfiguration: **启用 Spring 应用程序上下文的自动配置，试图猜测和配置您可能需要的bean。自动配置类通常采用基于你的classpath 和已经定义的 beans 对象进行应用。被 @EnableAutoConfiguration 注解的类所在的包有特定的意义，并且作为默认配置使用。通常推荐将 @EnableAutoConfiguration 配置在 root 包下，这样所有的子包、类都可以被查找到。

**@ComponentScan：**注解在类上，扫描标注了@Controller等注解的类，注册为bean 。@ComponentScan 为 @Configuration注解的类配置组件扫描指令。@ComponentScan 注解会自动扫描指定包下的全部标有 @Component注解的类，并注册成bean，当然包括 @Component下的子注解@Service、@Repository、@Controller。

**@Configuration: **注解在类上，表示这是一个IOC容器，相当于spring的配置文件，java配置的方式。 IOC容器的配置类一般与 @Bean 注解配合使用，用 @Configuration 注解类等价与 XML 中配置 beans，用@Bean 注解方法等价于 XML 中配置 bean。

**@Bean： **注解在方法上，声明当前方法返回一个Bean

**@Autowired：**按类型注入.默认属性required= true;当不能确定 Spring 容器中一定拥有某个类的Bean 时， 可以在需要自动注入该类 Bean 的地方可以使用 @Autowired\(required = false\)， 这等于告诉Spring：在找不到匹配Bean时也不抛出BeanCreationException 异常。@Autowired 和 @Qualifier 结合使用时，自动注入的策略就从 byType 转变byName 了。@Autowired可以对成员变量、方法以及构造函数进行注释，而 @Qualifier 的标注对象是成员变量、方法入参、构造函数入参。正是由于注释对象的不同，所以 Spring 不将 @Autowired 和 @Qualifier 统一成一个注释类，@Resource： 按名称装配

区别：@Resource默认按照名称方式进行bean匹配，@Autowired默认按照类型方式进行bean匹配

**@Resource:  **默认按照名称方式进行bean匹配

**@Service: **注解在类上，表示这是一个业务层bean

**@Controller：**注解在类上，表示这是一个控制层bean

**@RestController**：用于标注控制层组件\(如struts中的action\)，结合了 @ResponseBody和 @Controller 的注解

**@Repository: ** 注解在类上，表示这是一个数据访问层bean

**@Component：** 注解在类上，表示通用bean，value不写默认就是类名首字母小写

**@Scope：**注解在类上，描述spring容器如何创建Bean实例。

（1）singleton： 表示在spring容器中的单例，通过spring容器获得该bean时总是返回唯一的实例

（2）prototype：表示每次获得bean都会生成一个新的对象

（3）request：表示在一次http请求内有效（只适用于web应用）

（4）session：表示在一个用户会话内有效（只适用于web应用）

（5）globalSession：表示在全局会话内有效（只适用于web应用）

**@Value：**注解在变量上，从配置文件中读取。例如：@Value\(value = “\#{message}”\)

**@ConfigurationProperties：**赋值，将注解转换成对象。给对象赋值。车险项目：HttpClientSetting类

**@Profile：**注解在方法类上在不同情况下选择实例化不同的Bean特定环境下生效

**@ResponseBody**：表示该方法的返回结果直接写入HTTP response body中，一般在异步

获取数据时使用，用于构建RESTful的api。在使用@RequestMapping后，返回值通常解析

为跳转路径，加上@responsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP

response body中。比如异步获取json数据，加上@responsebody后，会直接返回json数

据。该注解一般会配合@RequestMapping一起使用。

**@PathVariable\(路径变量\)和@RequestParam: **

** **两者的作用都是将request里的参数的值绑定到contorl里的方法参数里的，区别在于，URL写法不同。当请求参数username不存在时会有异常发生,可以通过设置属性required=false解决,例如:

@RequestParam\(value="username",required=false\)

使用@RequestParam时，URL是这样的：[http://host:port/path?参数名=参数值](http://host:port/path?参数名=参数值)

使用@PathVariable时，URL是这样的：[http://host:port/path/参数值](http://host:port/path/参数值)

不写的时候也可以获取到参数值，但是必须名称对应。参数可以省略不写

```
RequestMapping(“user/get/mac/{macAddress}”)
public String getByMacAddress(@PathVariable String macAddress){
//do something;
}
```

**@RequestMapping：**

```
和请求报文是做对应的
params:指定request中必须包含某些参数值是，才让该方法处理。
headers:指定request中必须包含某些指定的header值，才能让该方法处理请求。
value:指定请求的实际地址，指定的地址可以是URI Template 模式
method:指定请求的method类型， GET、POST、PUT、DELETE等
consumes:指定处理请求的提交内容类型（Content-Type），如application/json,text/html;
produces:指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回

g: name  指定映射的名称:
@RequestMapping(method = RequestMethod.GET)
@RequestMapping(method = RequestMethod.POST)
@RequestMapping(method = RequestMethod.PUT)
@RequestMapping(method = RequestMethod.DELETE)
当然也可以使用
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping 这与上面的是一样的效果
```

**@EnableCaching：**注解是spring framework中的注解驱动的缓存管理功能。自spring版本3.1起加入了该注解。如果你使用了这个注解，那么你就不需要在XML文件中配置cache manager了。

**@suppresswarnings : **抑制警告

**@Modifying ：**如果是增，改，删加上此注解

1：方法的返回值应该是int，表示更新语句所影响的行数。

2：在调用的地方必须加事务，没有事务不能正常执行。

**@ExceptionHandler（Exception.class）**：用在方法上面表示遇到这个异常就执行以下方法。

## 注解优势

1.采用纯java代码，不在需要配置繁杂的xml文件

2.在配置中也可享受面向对象带来的好处

3.类型安全对重构可以提供良好的支持

4.减少复杂配置文件的同时亦能享受到springIoC容器提供的功能

## 参考

[https://blog.csdn.net/yitian\_66/article/details/80866571](https://blog.csdn.net/yitian_66/article/details/80866571)


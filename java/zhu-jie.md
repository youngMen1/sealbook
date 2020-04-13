# 1.SpringBoot注解

## 1.1.**注解\(annotations\)列表**

**@SpringBootApplication**：

包含了@ComponentScan、@Configuration和@EnableAutoConfiguration注解。其中@ComponentScan让spring Boot扫描到Configuration类并把它加入到程序上下文。

**@EnableAutoConfiguration: **启用 Spring 应用程序上下文的自动配置，试图猜测和配置您可能需要的bean。自动配置类通常采用基于你的classpath 和已经定义的 beans 对象进行应用。被 @EnableAutoConfiguration 注解的类所在的包有特定的意义，并且作为默认配置使用。通常推荐将 @EnableAutoConfiguration 配置在 root 包下，这样所有的子包、类都可以被查找到。

**@ComponentScan：**注解在类上，扫描标注了@Controller等注解的类，注册为bean 。@ComponentScan 为 @Configuration注解的类配置组件扫描指令。@ComponentScan 注解会自动扫描指定包下的全部标有 @Component注解的类，并注册成bean，当然包括 @Component下的子注解@Service、@Repository、@Controller。如果不设置basePackage的话 默认会扫描包的所有类，所以最好还是写上basePackage （@componentScan\({" ... "}）,减少加载时间。默认扫描\*\*/\*.class路径 比如这个注解在com.wuhulala 下面 ，那么会扫描这个包下的所有类还有子包的所有类,比如com.wuhulala.service包的应用

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

**@Transactional： **事务注解

**@Query： **自定义查询语句 JPQL

## 1.2.**JPA注解**

**@Entity**：@Table\(name=”“\)：表明这是一个实体类。一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，@Table可以省略

**@MappedSuperClass：**用在确定是父类的entity上。父类的属性子类可以继承。

**@NoRepositoryBean：** 一般用作父类的repository，有这个注解，spring不会去实例化该repository。

**@Column**：

```
通过@Column注解设置，包含的设置如下
name：数据库表字段名
unique：是否唯一
nullable：是否可以为空
Length:长度
inserttable：是否可以插入
updateable：是否可以更新
columnDefinition: 定义建表时创建此列的DDL
secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字。
@Column(name = "user_code", nullable = false, length=32)//设置属性userCode对应的字段为user_code，长度为32，非空
private String userCode;
//设置属性wages对应的字段为user_wages，12位数字可保留两位小数，可以为空
@Column(name = "user_wages", nullable = true, precision=12,scale=2)
private double wages;
```

**@Id**：表示该属性为主键。

**@GeneratedValue\(strategy = GenerationType.SEQUENCE,generator = “repair\_seq”\)**：表示主键生成策略是sequence（可以为Auto、IDENTITY、native等，Auto表示可在多个数据库间切换），指定sequence的名字是repair\_seq。

**@SequenceGeneretor\(name = “repair\_seq”, sequenceName = “seq\_repair”,allocationSize = 1\)**：name为sequence的名称，以便使用，sequenceName为数据库的sequence名称，两个名称可以一致。

**@Transient**：表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic。@Basic\(fetch=FetchType.LAZY\)：标记可以指定实体属性的加载方式

**@JsonIgnore**：作用是json序列化时将Java bean中的一些属性忽略掉,序列化和反序列化都受影响。

**@JoinColumn（name=”loginId”）：**一对一：本表中指向另一个表的外键。一对多：另一个表指向本表的外键。

**@OneToOne、@OneToMany、@ManyToOne**：对应hibernate配置文件中的一对一，一对多，多对一。

**@GeneratedValue：用于标注主键的生成策略，通过 strategy 属性指定。默认情况下，JPA 自动选择一个最适合底层数据库的主键生成策略：SqlServer 对应 identity，MySQL 对应 auto increment。 在 javax.persistence.GenerationType 中定义了以下几种可供选择的策略：**

IDENTITY：采用数据库 ID自增长的方式来自增主键字段，Oracle 不支持这种方式；

AUTO： JPA自动选择合适的策略，是默认选项；

SEQUENCE：通过序列产生主键，通过 @SequenceGenerator 注解指定序列名，MySql 不支持这种方式

TABLE：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。

**@Temporal\(TemporalType.DATE\)：**设置为时间类型     例如：private Date joinDate;

## 1.3.**全局异常处理**

**@ExceptionHandler（Exception.class）**：用在方法上面表示遇到这个异常就执行以下方法。

**@ControllerAdvice：**包含@Component。可以被扫描到。统一处理异常。

## 1.4.其他注解

**@Async和 @EnableAsync：**

@EnableAsync注解的意思是可以异步执行，就是开启多线程的意思。可以标注在方法、类上。为了让@Async注解能够生效，需要在Spring Boot的主程序中配置@EnableAsync，@Async所修饰的函数不要定义为static类型，这样异步调用不会生效

**@Mapper和@MapperScan：**

Mapper类上面添加注解@Mapper，这种方式要求每一个mapper类都需要添加此注解

使用@MapperScan可以指定要扫描的Mapper类的包的路径（@MapperScan\("com.demo.\*.mapper"\)\|\| @MapperScan\("com.test.\*.mapper", "com.demo.\*.mapper"\)）

**@LoadBalanced：**

Spring Cloud的commons模块提供了一个@LoadBalanced注解，方便我们对RestTemplate添加一个LoadBalancerClient，以实现客户端负载均衡。通过源码可以发现这是一个标记注解,我们可以通过ribbon实现客户端的负载均衡功能。

**@MappedSuperclass：**

1.@MappedSuperclass 注解使用在父类上面，是用来标识父类的

2.@MappedSuperclass 标识的类表示其不能映射到数据库表，因为其不是一个完整的实体类，但是它所拥有的属性能够映射在其子类对用的数据库表中

3.@MappedSuperclass 标识的类不能再有@Entity或@Table注解

**1）数据库查询**

**@PostLoad事件在下列情况下触发：**

执行EntityManager.find\(\)或getreference\(\)方法载入一个实体后。

执行JPQL查询后。

EntityManager.refresh\(\)方法被调用后。

**2）数据库插入**

**@PrePersist和@PostPersist事件在实体对象插入到数据库的过程中发生：**

@PrePersist事件在调用persist\(\)方法后立刻发生，此时的数据还没有真正插入进数据库。

@PostPersist事件在数据已经插入进数据库后发生。

**3）数据库更新**

**@PreUpdate和@PostUpdate事件的触发由更新实体引起：**

@PreUpdate事件在实体的状态同步到数据库之前触发，此时的数据还没有真正更新到数据库。

@PostUpdate事件在实体的状态同步到数据库之后触发，同步在事务提交时发生。

**4）数据库删除**

**@PreRemove和@PostRemove事件的触发由删除实体引起：**

@PreRemove事件在实体从数据库删除之前触发，即在调用remove\(\)方法删除时发生，此时的数据还没有真正从数据库中删除。

@PostRemove事件在实体从数据库中删除后触发。

# 2.lombok插件注解

**@Setter：**生成setter方法,final变量不包含

**@Getter：**生成getter方法,final变量不包含

**@NoArgsConstructor：**生成一个无参数的构造方法

**@AllArgsConstructor：**生成全部参数构造

**@RequiredArgsConstructor：** 会生成一个包含常量，和标识了NotNull的变量的构造方法。生成的构造方法是私有的private。

主要使用前两个注解，这样就不需要自己写构造方法，代码简洁规范,将标记为@NoNull的属性生成一个构造器,如果运行中标记为@NoNull的属性为null,会抛出空指针异常。

**@ToString：**生成所有属性的toString\(\)方法

**@EqualsAndHashCode：**生成equals\(\)方法和hashCode方法

**@Data\(常用\)：**@Data直接修饰POJO or beans， getter所有的变量，setter所有不为final的变量。如果你不需要默认的生成方式，直接填写你需要的annotation的就可以了。默认生成的所有的annotation都是public的，如果需要不同权限修饰符可以使用AccessLevel.NONE选项。当然@Data 也可以使用staticConstructor选项生成一个静态方法。

等于:@Setter+@Getter+@EqualsAndHashCode+@NoArgsConstructor

**@Builder：**构造Builder模式的结构。通过内部类Builder\(\)进行构建对象。

**@Value：**与@Data相对应的@Value， 两个annotation的主要区别就是如果变量不加@NonFinal ，@Value会给所有的弄成final的。当然如果是final的话，就没有set方法了。

**@Synchronized：**同步方法，加个同步锁

**@Cleanup 和@SneakyThrows：**自动调用close方法关闭资源。

**@Cleanup：**

注释在引用变量前：自动回收资源 默认调用close方法

@Cleanup\("dispose"\) org.eclipse.swt.widgets.CoolBar bar = new CoolBar\(parent, 0\);

@Cleanup InputStream in = new FileInputStream\(args\[0\]\);

@Cleanup OutputStream out = new FileOutputStream\(args\[1\]\);

**@Log4j ：**注解在类上；为类提供一个 属性名为log 的 log4j 日志对象

**@NonNull：**注解在参数上 如果该参数为null 会throw new NullPointerException\(参数名\);

**@Slf4j：**使用log打印日志

**@Accessors：**注解也用在lombok不同的寻找getters的方法中，例如@EqualsAndHashCode\(不懂\)如果提供了前缀列表，属性名没有一个以其中的一个前缀开头，则属性会被lombok完全忽略掉，并且会产生一个警告。

# 3.swagger注解

**@Api\(\)**

用于类；表示标识这个类是swagger的资源

tags–表示说明

value–也是说明，可以使用tags替代

```
@Api(value="用户controller",tags={"用户操作接口"})
```

**@ApiOperation\(\)**

用于方法；表示一个http请求的操作

value用于方法描述

notes用于提示内容

tags可以重新分组（视情况而用）

```
@ApiOperation(value="获取用户信息",tags={"获取用户信息copy"},notes="注意问题点")
@PostMapping("/updateUserInfo")
public int updateUserInfo(@RequestBody @ApiParam(name="用户对象",value="传入json格式",required=true) User user){

   int num = userService.updateUserInfo(user);
   return num;
}
```

**@ApiParam\(\)**

用于方法，参数，字段说明；表示对参数的添加元数据（说明或是否必填等）

name–参数名

value–参数说明

required–是否必填

```
@Api(value="用户controller",tags={"用户操作接口"})
@RestController
public class UserController {
     @ApiOperation(value="获取用户信息",tags={"获取用户信息copy"},notes="注意问题点")
     @GetMapping("/getUserInfo")
     public User getUserInfo(@ApiParam(name="id",value="用户id",required=true) Long id,@ApiParam(name="username",value="用户名") String username) {
     // userService可忽略，是业务逻辑
      User user = userService.getUserInfo();

      return user;
  }
}
```

**@ApiModel\(\)**

用于类 ；表示对类进行说明，用于参数用实体类接收

value–表示对象名

description–描述

都可省略

**@ApiModelProperty\(\)**

用于方法，字段； 表示对model属性的说明或者数据操作更改

value–字段说明

name–重写属性名字

dataType–重写属性类型

required–是否必填

example–举例说明

hidden–隐藏

```
@ApiModel(value="user对象",description="用户对象user")
public class User implements Serializable{
    private static final long serialVersionUID = 1L;
     @ApiModelProperty(value="用户名",name="username",example="xingguo")
     private String username;
     @ApiModelProperty(value="状态",name="state",required=true)
      private Integer state;
      private String password;
      private String nickName;
      private Integer isDeleted;

      @ApiModelProperty(value="id数组",hidden=true)
      private String[] ids;
      private List<String> idList;
     //省略get/set
}
```

**@ApiIgnore\(\)**

用于类或者方法上；可以不被swagger显示在页面上  
比较简单, 这里不做举例

**@ApiImplicitParam\(\)**

用于方法表示单独的请求参数  
**@ApiImplicitParams\(\)**

用于方法；包含多个 @ApiImplicitParam  
name–参数ming  
value–参数说明  
dataType–数据类型  
paramType–参数类型  
example–举例说明

```
@ApiOperation("查询测试")
  @GetMapping("select")
  //@ApiImplicitParam(name="name",value="用户名",dataType="String", paramType = "query")
  @ApiImplicitParams({
  @ApiImplicitParam(name="name",value="用户名",dataType="string", paramType = "query",example="xingguo"),
  @ApiImplicitParam(name="id",value="用户id",dataType="long", paramType = "query")})
  public void select(){

  }
```

**@ApiImplicitParam：**用在@ApiImplicitParams注解中，指定一个请求参数的各个方面

* paramType：参数放在哪个地方
* name：参数代表的含义
* value：参数名称
* dataType： 参数类型，有String/int，无用
* required ： 是否必要
* defaultValue：参数的默认值

![img](/static/image/998342-20171129203814573-235322774.png)

**@ApiImplicitParams：**用在方法上包含一组参数说明；

**@ApiResponses：**用于表示一组响应；

**@ApiResponse：**用在@ApiResponses中，一般用于表达一个错误的响应信息；

* code： 响应码\(int型\)，可自定义
* message：状态码对应的响应信息

# 4.元注解

## 4.1.Annotation\(注解\)

从JDK 1.5开始, Java增加了对元数据\(MetaData\)的支持，也就是 Annotation\(注解\)。

注解其实就是代码里的特殊标记，它用于替代配置文件：传统方式通过配置文件告诉类如何运行，有了注解技术后，开发人员可以通过注解告诉类如何运行。在Java技术里注解的典型应用是：可以通过反射技术去得到类里面的注解，以决定怎么去运行类。

注解可以标记在包、类、属性、方法，方法参数以及局部变量上，且同一个地方可以同时标记多个注解。

```
// 抑制编译期的未指定泛型、未使用和过时警告
@SuppressWarnings({ "rawtypes", "unused", "deprecation" })
// 重写
@Override
```

## 4.2.meta-annotation（元注解）

* 注解参数的可支持数据类型：  
所有基本数据类型（int,float,boolean,byte,double,char,long,short\)  
String类型  
Class类型  
enum类型  
Annotation类型  
以上所有类型的数组

除了直接使用JDK 定义好的注解，我们还可以自定义注解，在JDK 1.5中提供了4个标准的用来对注解类型进行注解的注解类，我们称之为 meta-annotation（元注解），他们分别是：

  * @Target

  * @Retention

  * @Documented

  * @Inherited

### 4.2.1.@Target注解
Target注解的作用是：描述注解的使用范围（即：被修饰的注解可以用在什么地方）。

Target注解用来说明那些被它所注解的注解类可修饰的对象范围：注解可以用于修饰 packages、types（类、接口、枚举、注解类）、类成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数），在定义注解类时使用了@Target 能够更加清晰的知道它能够被用来修饰哪些对象，它的取值范围定义在ElementType 枚举中。

```
public enum ElementType {
 
    TYPE, // 类、接口、枚举类
 
    FIELD, // 成员变量（包括：枚举常量）
 
    METHOD, // 成员方法
 
    PARAMETER, // 方法参数
 
    CONSTRUCTOR, // 构造方法
 
    LOCAL_VARIABLE, // 局部变量
 
    ANNOTATION_TYPE, // 注解类
 
    PACKAGE, // 可用于修饰：包
 
    TYPE_PARAMETER, // 类型参数，JDK 1.8 新增
 
    TYPE_USE // 使用类型的任何地方，JDK 1.8 新增
}

```
###4.2.2.@Retention注解

Reteniton注解的作用是：描述注解保留的时间范围（即：被描述的注解在它所修饰的类中可以被保留到何时） 。

Reteniton注解用来限定那些被它所注解的注解类在注解到其他类上以后，可被保留到何时，一共有三种策略，定义在RetentionPolicy枚举中。


```
public enum RetentionPolicy {
 
    SOURCE,    // 源文件保留
    CLASS,       // 编译期保留，默认值
    RUNTIME   // 运行期保留，可通过反射去获取注解信息
}

```
为了验证应用了这三种策略的注解类有何区别，分别使用三种策略各定义一个注解类做测试。


```
@Retention(RetentionPolicy.SOURCE)
public @interface SourcePolicy {
}
@Retention(RetentionPolicy.CLASS)
public @interface ClassPolicy {
}
@Retention(RetentionPolicy.RUNTIME)
public @interface RuntimePolicy {
}
用定义好的三个注解类分别去注解一个方法。
public class RetentionTest {
	@SourcePolicy
	public void sourcePolicy() {
	}
 
	@ClassPolicy
	public void classPolicy() {
	}
 
	@RuntimePolicy
	public void runtimePolicy() {
	}
}

```
![img](/static/image/20180325084649363)
如图所示，通过执行 javap -verbose RetentionTest命令获取到的RetentionTest 的 class 字节码内容如下。


```
{
  public retention.RetentionTest();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public void sourcePolicy();
    flags: ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 7: 0

  public void classPolicy();
    flags: ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 11: 0
    RuntimeInvisibleAnnotations:
      0: #11()

  public void runtimePolicy();
    flags: ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 15: 0
    RuntimeVisibleAnnotations:
      0: #14()
}
```
从 RetentionTest 的字节码内容我们可以得出以下两点结论：
           1. 编译器并没有记录下 sourcePolicy() 方法的注解信息； 
           2. 编译器分别使用了 RuntimeInvisibleAnnotations 和 RuntimeVisibleAnnotations 属性去记录了classPolicy()方法 和 runtimePolicy()方法 的注解信息；  

### 4.2.3.@Documented注解
Documented注解的作用是：**描述在使用 javadoc 工具为类生成帮助文档时是否要保留其注解信息。**

为了验证Documented注解的作用到底是什么，我们创建一个带有 @Documented 的自定义注解类。 


```
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;
 
@Documented
@Target({ElementType.TYPE,ElementType.METHOD})
public @interface MyDocumentedtAnnotation {
 
	public String value() default "这是@Documented注解为文档添加的注释";
}


```
再创建一个 MyDocumentedTest 类。  


```
@MyDocumentedtAnnotation
public class MyDocumentedTest {
 
	@Override
	@MyDocumentedtAnnotation
	public String toString() {
		return this.toString();
	}
}

```
接下来，使用以下命令为 MyDocumentedTest 类生成帮助文档。  
![img](/static/image/20180325085153442)
命令执行完成之后，会在当前目录下生成一个 doc 文件夹，其内包含以下文件。 
![img](/static/image/20180325085233066)
查看 index.html 帮助文档，可以发现在类和方法上都保留了 MyDocumentedtAnnotation 注解信息。 
![img](static/image/20180325085330837)
修改 MyDocumentedtAnnotation 注解类，去掉上面的 @Documented 注解。 


```
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;
 
@Target({ElementType.TYPE,ElementType.METHOD})
public @interface MyDocumentedtAnnotation {
 
	public String value() default "这是@Documented注解为文档添加的注释";
}

```
重新生成帮助文档，此时类和方法上的 MyDocumentedtAnnotation 注解信息都不见了。 
![img](/static/image/20180325085507391)
### 4.2.4.@Inherited注解


# 5.注解优势

1.采用纯java代码，不在需要配置繁杂的xml文件

2.在配置中也可享受面向对象带来的好处

3.类型安全对重构可以提供良好的支持

4.减少复杂配置文件的同时亦能享受到springIoC容器提供的功能

## 6.参考

[https://blog.csdn.net/yitian\_66/article/details/80866571](https://blog.csdn.net/yitian_66/article/details/80866571)

[https://blog.csdn.net/u013225178/article/details/80721799](https://blog.csdn.net/u013225178/article/details/80721799)

swagger： [https://www.jianshu.com/p/12f4394462d5](https://www.jianshu.com/p/12f4394462d5)


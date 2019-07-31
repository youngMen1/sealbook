| @Component | 基本注解, 标识了一个受 Spring 管理的组件 |
| :--- | :--- |
| @Respository | 标识持久层组件 |
| @Service | 标识服务层\(业务层\)组件 |
| @Controller | 标识表现层组件 |
| @Autowired | 注解自动装配**具有兼容类型**的单个 Bean属性 |
| @Resource | @Resource 注解要求提供一个 Bean 名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为 Bean 的名称 |
| @Bean | @Bean注解告诉 Spring，一个带有 @Bean 的注解方法将返回一个对象，该对象应该被注册为在 Spring 应用程序上下文中的 bean。 |
| @Qualifier | 可能会有这样一种情况，当你创建多个具有相同类型的 bean 时，并且想要用一个属性只为它们其中的一个进行装配，在这种情况下，你可以使用**@Qualifier**注释和**@Autowired**注释通过指定哪一个真正的 bean 将会被装配来消除混乱。下面显示的是使用 @Qualifier 注释的一个示例。 |
| 1 | 1 |
| 1 | 1 |
| 1 | 1 |

## @Configuration

**注意**：@Configuration注解的配置类有如下要求：

1. @Configuration不可以是final类型；
2. @Configuration不可以是匿名类；
3. 嵌套的configuration必须是静态类。

一、用@Configuration加载spring  
1.1、@Configuration配置spring并启动spring容器  
1.2、@Configuration启动容器+@Bean注册Bean  
1.3、@Configuration启动容器+@Component注册Bean  
1.4、**使用 AnnotationConfigApplicationContext 注册 AppContext 类的两种方法  
1.5、配置Web应用程序\(web.xml中配置AnnotationConfigApplicationContext\)**

二、组合多个配置类  
2.1、在@configuration中引入spring的xml配置文件  
2.2、在@configuration中引入其它注解配置  
2.3、@configuration嵌套（嵌套的Configuration必须是静态类）  
三、@EnableXXX注解  
四、@Profile逻辑组配置  
五、使用外部变量

---

## **@ComponentScan**

自定扫描路径下边带有@Controller，@Service，@Repository，@Component注解加入spring容器  
通过includeFilters加入扫描路径下没有以上注解的类加入spring容器  
通过excludeFilters过滤出不用加入spring容器的类  
自定义增加了@Component注解的注解方式

---

# @ProtertySource

@PropertySouce是spring3.1开始引入的基于java config的注解。

通过@PropertySource注解将properties配置文件中的值存储到Spring的 Environment中，Environment接口提供方法去读取配置文件中的值，参数是properties文件中定义的key值。

---

## 常见AspectJ的注解：

1.  @Before
   –
   方法执行前运行
2.  @After
   –
   运行在方法返回结果后
3.  @AfterReturning
   –
   运行在方法返回一个结果后，在拦截器返回结果。
4.  @AfterThrowing
   –
   运行方法在抛出异常后，
5.  @Around
   –
   围绕方法执行运行，结合以上这三个通知。



  


  



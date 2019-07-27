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

通过@PropertySource注解将properties配置文件中的值存储到Spring的 Environment中，Environment接口提供方法去读取配置文件中的值，参数是properties文件中定义的key值。




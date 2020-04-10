# 1.SpringBoot注解

## 1.1.**注解\(annotations\)列表**

**@SpringBootApplication**：

包含了@ComponentScan、@Configuration和@EnableAutoConfiguration注解。其中@ComponentScan让spring Boot扫描到Configuration类并把它加入到程序上下文。

**@Configuration:**

注解在类上，表示这是一个IOC容器，相当于spring的配置文件，java配置的方式。 IOC容器的配置类一般与 @Bean 注解配合使用，用 @Configuration 注解类等价与 XML 中配置 beans，用@Bean 注解方法等价于 XML 中配置 bean。

@Bean： 注解在方法上，声明当前方法返回一个Bean

## 注解优势

1.采用纯java代码，不在需要配置繁杂的xml文件

2.在配置中也可享受面向对象带来的好处

3.类型安全对重构可以提供良好的支持

4.减少复杂配置文件的同时亦能享受到springIoC容器提供的功能


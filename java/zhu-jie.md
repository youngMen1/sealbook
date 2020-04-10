# 1.SpringBoot注解

## 1.1.**注解\(annotations\)列表**

**@SpringBootApplication**：

包含了@ComponentScan、@Configuration和@EnableAutoConfiguration注解。其中@ComponentScan让spring Boot扫描到Configuration类并把它加入到程序上下文。

**@Configuration:**

等同于spring的XML配置文件；使用Java代码可以检查类型安全。

## 注解优势

使用注解的优势：

     1.采用纯java代码，不在需要配置繁杂的xml文件

     2.在配置中也可享受面向对象带来的好处

     3.类型安全对重构可以提供良好的支持

     4.减少复杂配置文件的同时亦能享受到springIoC容器提供的功能




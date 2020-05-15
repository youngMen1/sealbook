为什么要写spring.factories文件

在spring-boot项目中pom文件里面添加的依赖中的bean是如何注册到spring-boot项目的spring容器中的呢？”，不难得出spring.factories文件是帮助spring-boot项目包以外的bean（即在pom文件中添加依赖中的bean）注册到spring-boot项目的spring容器的结论。由于@ComponentScan注解只能扫描spring-boot项目包内的bean并注册到spring容器中，因此需要@EnableAutoConfiguration注解来注册项目包外的bean。而spring.factories文件，则是用来记录项目包外需要注册的bean类名。

# 参考

原文链接：[https://blog.csdn.net/SkyeBeFreeman/article/details/96291283](https://blog.csdn.net/SkyeBeFreeman/article/details/96291283)


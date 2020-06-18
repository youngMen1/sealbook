# 1.Logback配置文件这么写，TPS提高10倍

## 1.1.配置文件logback-spring.xml

SpringBoot工程自带logback和slf4j的依赖，所以重点放在编写配置文件上，需要引入什么依赖，日志依赖冲突统统都不需要我们管了。logback框架会默认加载classpath下命名为logback-spring或logback的配置文件。将所有日志都存储在一个文件中文件大小也随着应用的运行越来越大并且不好排查问题，正确的做法应该是将error日志和其他日志分开，并且不同级别的日志根据时间段进行记录存储。




# 1.Logback配置文件这么写，TPS提高10倍

## 1.1.配置文件logback-spring.xml

SpringBoot工程自带logback和slf4j的依赖，所以重点放在编写配置文件上，需要引入什么依赖，日志依赖冲突统统都不需要我们管了。logback框架会默认加载classpath下命名为logback-spring或logback的配置文件。将所有日志都存储在一个文件中文件大小也随着应用的运行越来越大并且不好排查问题，正确的做法应该是将error日志和其他日志分开，并且不同级别的日志根据时间段进行记录存储。

### 同步配置
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <property resource="logback.properties"/>
    <appender name="CONSOLE-LOG" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>[%d{yyyy-MM-dd' 'HH:mm:ss.sss}] [%C] [%t] [%L] [%-5p] %m%n</pattern>
        </layout>
    </appender>
    <!--获取比info级别高(包括info级别)但除error级别的日志-->
    <appender name="INFO-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>DENY</onMatch>
            <onMismatch>ACCEPT</onMismatch>
        </filter>
        <encoder>
            <pattern>[%d{yyyy-MM-dd' 'HH:mm:ss.sss}] [%C] [%t] [%L] [%-5p] %m%n</pattern>
        </encoder>

        <!--滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--路径-->
            <fileNamePattern>${LOG_INFO_HOME}//%d.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    </appender>
    <appender name="ERROR-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <encoder>
            <pattern>[%d{yyyy-MM-dd' 'HH:mm:ss.sss}] [%C] [%t] [%L] [%-5p] %m%n</pattern>
        </encoder>
        <!--滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--路径-->
            <fileNamePattern>${LOG_ERROR_HOME}//%d.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    </appender>

    <root level="info">
        <appender-ref ref="CONSOLE-LOG" />
        <appender-ref ref="INFO-LOG" />
        <appender-ref ref="ERROR-LOG" />
    </root>
</configuration>

```
### 部分标签说明
<root>标签，必填标签，用来指定最基础的日志输出级别
* <appender-ref>标签，添加append
<append>标签，通过使用该标签指定日志的收集策略
name属性指定appender命名
class属性指定输出策略，通常有两种，控制台输出和文件输出，文件输出就是将日志进行一个持久化。ConsoleAppender将日志输出到控制台
<filter>标签，通过使用该标签指定过滤策略
* <level>标签指定过滤的类型
<encoder>标签，使用该标签下的<pattern>标签指定日志输出格式
<rollingPolicy>标签指定收集策略，比如基于时间进行收集
* <fileNamePattern>标签指定生成日志保存地址
通过这样配置已经实现了分类分天手机日志的目标了

## 1.2.logback 高级特性异步输出日志
之前的日志配置方式是基于同步的，每次日志输出到文件都会进行一次磁盘IO。采用异步写日志的方式而不让此次写日志发生磁盘IO，阻塞线程从而造成不必要的性能损耗。异步输出日志的方式很简单，添加一个基于异步写日志的appender，并指向原先配置的appender即可

### 异步输出日志
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
<property resource="logback.properties"/>
<appender name="CONSOLE-LOG" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>[%d{yyyy-MM-dd' 'HH:mm:ss.sss}] [%C] [%t] [%L] [%-5p] %m%n</pattern>
    </layout>
</appender>

<!--异步输出日志-->

<!--获取比info级别高(包括info级别)但除error级别的日志-->
<appender name="INFO-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>ERROR</level>
        <onMatch>DENY</onMatch>
        <onMismatch>ACCEPT</onMismatch>
    </filter>
    <encoder>
        <pattern>[%d{yyyy-MM-dd' 'HH:mm:ss.sss}] [%C] [%t] [%L] [%-5p] %m%n</pattern>
    </encoder>

    <!--滚动策略-->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!--路径-->
        <fileNamePattern>${LOG_INFO_HOME}//%d.log</fileNamePattern>
        <maxHistory>30</maxHistory>
    </rollingPolicy>
</appender>
<appender name="ERROR-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>ERROR</level>
    </filter>
    <encoder>
        <pattern>[%d{yyyy-MM-dd' 'HH:mm:ss.sss}] [%C] [%t] [%L] [%-5p] %m%n</pattern>
    </encoder>
    <!--滚动策略-->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!--路径-->
        <fileNamePattern>${LOG_ERROR_HOME}//%d.log</fileNamePattern>
        <maxHistory>30</maxHistory>
    </rollingPolicy>
</appender>
<!-- 异步输出 -->
<appender name="ASYNC-INFO" class="ch.qos.logback.classic.AsyncAppender">
    <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
    <discardingThreshold>0</discardingThreshold>
    <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
    <queueSize>256</queueSize>
    <!-- 添加附加的appender,最多只能添加一个 -->
    <appender-ref ref="INFO-LOG"/>
</appender>

<appender name="ASYNC-ERROR" class="ch.qos.logback.classic.AsyncAppender">
    <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
    <discardingThreshold>0</discardingThreshold>
    <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
    <queueSize>256</queueSize>
    <!-- 添加附加的appender,最多只能添加一个 -->
    <appender-ref ref="ERROR-LOG"/>
</appender>

<root level="info">
    <appender-ref ref="CONSOLE-LOG"/>
    <appender-ref ref="ASYNC-INFO"/>
    <appender-ref ref="ASYNC-ERROR"/>
</root>
</configuration>

```
## 1.3.异步日志输出原理



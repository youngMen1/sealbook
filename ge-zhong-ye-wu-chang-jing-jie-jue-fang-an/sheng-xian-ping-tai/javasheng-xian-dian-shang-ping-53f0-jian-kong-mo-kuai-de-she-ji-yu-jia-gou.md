# Java生鲜电商平台-监控模块的设计与架构

说明：Java开源生鲜电商平台-监控模块的设计与架构，我们谈到监控，一般设计到两个方面的内容：

1.服务器本身的监控。(比如：linux服务器的CPU，内存，磁盘IO等监控)

2.业务系统的监控.  (比如：业务系统性能的监控，SQL语句的监控，请求超时的监控，用户输入的监控，整个请求过程时间的监控，优化等等)

1. 服务器本身的监控

说明：由于Java开源生鲜电商平台采用的是阿里云的linux CentOS服务器，由于阿里云本身是有监控预警的，但是我们不可能时刻去看，最好有集成自己的系统监控，

最终在各种系统对比的过程中，选择了netdata这个工具，当然有一些软件比如：zabbix,negios等等都是可以的，但是我们服务器压力不算大，最终采用了更加轻量级的解决方案。

相关的安装与使用，大家自行百度处理，我这边就不列举出来了。

**以下是相关的实际运营截图：**


2.业务监控
说明：任何一个业务系统都需要采用业务监控，抛异常，有error日志，短信预警，推送等等

1.Java内存
2.JavaCPU使用情况
3.用户Session数量
4.JDBC连接数
5.http请求、sql请求、jsp页面与业务接口方法（EJB3、Spring、 Guice）的执行数量，平均执行时间，错误百分比等

最终，业务代码中采用了Spring AOP进行日志拦截，把请求方法超过了1500秒的方法进行了error日志的输出：



```
import org.apache.commons.lang.time.StopWatch;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
/**
 * 声明一个切面,记录每个Action的执行时间
 */
@Aspect
@Component
public class LogAspect {
    
    private static final Logger logger=LoggerFactory.getLogger(LogAspect.class);
    
    /**
     * 切入点：表示在哪个类的哪个方法进行切入。配置有切入点表达式
     */
    @Pointcut("execution(* com.netcai.admin.controller.*.*.*(..))")
    public void pointcutExpression() {
        logger.debug("配置切入点");
    }
    
    /**
     * 1 前置通知
     * @param joinPoint
     */
    @Before("pointcutExpression()")
    public void beforeMethod(JoinPoint joinPoint) {
        logger.debug("前置通知执行了");
    }
    
    /**
     * 2 后置通知
     * 在方法执行之后执行的代码. 无论该方法是否出现异常
     */
    @After("pointcutExpression()") 
    public void afterMethod(JoinPoint joinPoint) {
        logger.debug("后置通知执行了，有异常也会执行");
    }
    
    /**
     * 3 返回通知
     * 在方法法正常结束受执行的代码
     * 返回通知是可以访问到方法的返回值的!
     * @param joinPoint
     * @param returnValue
     */
    @AfterReturning(value = "pointcutExpression()", returning = "returnValue")
    public void afterRunningMethod(JoinPoint joinPoint, Object returnValue) {
        logger.debug("返回通知执行，执行结果：" + returnValue);
    }
    /**
     * 4 异常通知
     * 在目标方法出现异常时会执行的代码.
     * 可以访问到异常对象; 且可以指定在出现特定异常时在执行通知代码
     * @param joinPoint
     * @param e
     */
    @AfterThrowing(value = "pointcutExpression()", throwing = "e")
    public void afterThrowingMethod(JoinPoint joinPoint, Exception e)
    {
        logger.debug("异常通知, 出现异常 " + e);
    }
    
    /**
     * 环绕通知需要携带 ProceedingJoinPoint 类型的参数. 
     * 环绕通知类似于动态代理的全过程: ProceedingJoinPoint 类型的参数可以决定是否执行目标方法.
     * 且环绕通知必须有返回值, 返回值即为目标方法的返回值
     */
    @Around("pointcutExpression()")
    public Object aroundMethod(ProceedingJoinPoint pjd)
    {
        StopWatch clock = new StopWatch();
        //返回的结果
        Object result = null;
        //方法名称
        String className=pjd.getTarget().getClass().getName();
        
        String methodName = pjd.getSignature().getName();
        
        try 
        {
            // 计时开始
            clock.start(); 
            //前置通知
            //执行目标方法
            result = pjd.proceed();
            //返回通知
            clock.stop();
        } catch (Throwable e) 
        {
            //异常通知
            e.printStackTrace();
        }
        //后置通知
        if(!methodName.equalsIgnoreCase("initBinder"))
        {
            long constTime=clock.getTime();
            
            logger.info("["+className+"]"+"-" +"["+methodName+"]"+" 花费时间： " +constTime+"ms");
            
            if(constTime>500)
            {
                logger.error("["+className+"]"+"-" +"["+methodName+"]"+" 花费时间过长，请检查: " +constTime+"ms");
            }
        }
        return result;
    }
}
```
补充说明：这个方法记录那个类，那个方法执行的时间多少，超过设置的阀值，那么就打印error日志，需要我们每天进行查看与针对性的优化。

3.对于整个业务线的监控，我们采用了另外一种开源的监控：javamelody

相关的配置与处理如下：

POM文件中设置：


```
<!-- 系统监控 -->
        <dependency>
            <groupId>net.bull.javamelody</groupId>
            <artifactId>javamelody-core</artifactId>
            <version>1.68.1</version>
        </dependency>

        <dependency>
            <groupId>org.jrobin</groupId>
            <artifactId>jrobin</artifactId>
            <version>1.5.9</version>
        </dependency>
```

web.xml文件中处理

```
<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:config/applicationContext.xml
            classpath*:net/bull/javamelody/monitoring-spring.xml
            classpath*:net/bull/javamelody/monitoring-spring-datasource.xml
            classpath*:net/bull/javamelody/monitoring-spring-aspectj.xml
        </param-value>
    </context-param>
```


```
<!--javamelody -->
    <filter>
        <filter-name>monitoring</filter-name>
        <filter-class>net.bull.javamelody.MonitoringFilter</filter-class>
        <async-supported>true</async-supported>
        <init-param>
            <param-name>logEnabled</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>monitoring</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <listener>
        <listener-class>net.bull.javamelody.SessionListener</listener-class>
    </listener>
```

最终运营效果如下：

641237-20180522092446489-123916981.png




# 1.基本介绍

![](/static/image/20171101213339213.png)

## 1.1.Spring事务的原理

Spring 事务管理分为编码式和声明式的两种方式。编程式事务指的是通过编码方式实现事务；声明式事务基于 AOP,将具体业务逻辑与事务处理解耦。声明式事务管理使业务代码逻辑不受污染, 因此在实际使用中声明式事务用的比较多。声明式事务有两种方式，一种是在配置文件中做相关的事务规则声明，另一种是基于@Transactional 注解的方式。

使用@Transactional的相比传统的我们需要手动开启事务，然后提交事务来说。它提供如下方便：

根据你的配置，设置是否自动开启事务

自动提交事务或者遇到异常自动回滚

声明式事务（@Transactional\)基本原理如下：

配置文件开启注解驱动，在相关的类和方法上通过注解@Transactional标识。

spring 在启动的时候会去解析生成相关的bean，这时候会查看拥有相关注解的类和方法，并且为这些类和方法生成代理，并根据@Transaction的相关参数进行相关配置注入，这样就在代理中为我们把相关的事务处理掉了（开启正常提交事务，异常回滚事务）。

真正的数据库层的事务提交和回滚是通过binlog\(规档日志\)或者redo log\(重做日志\)实现的。

### 1.1.1.事务管理的方式： {#事务管理的方式}

* Spring（Spring Framework 提供对事务管理的抽象接口） 支持两种事务管理方式：

  * 编程式事务管理：使用TransactionTemplate或PlatformTransactionManager实现
  * 声明式事务管理：建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务（此处取决于事务的传播行为），在执行完目标方法之后根据执行情况提交或者回滚事务（执行成功则提交，失败则进行实物的回滚）
  * 编程式事务管理优势：可以控制事务的粒度，最细粒度到代码块级别；
  * 声明式实物管理优势：在方法外进行声明，事务控制的代码不会与业务逻辑代码混在一起，最细粒度到方法级别（解决方法：可以将需要进行事务管理的代码块独立为方法，通过方法间调用实现）；符合spring倡导的非侵入式的开发方式，即业务处理逻辑代码与事务管理代码不放在一起

### 1.1.2.属性

### rollbackFor、rollbackForClassName、noRollbackFor、noRollbackForClassName

rollbackFor、rollbackForClassName用于设置那些异常需要回滚；

noRollbackFor、noRollbackForClassName用于设置那些异常不需要回滚。他们就是在设置事务的回滚规则。

## 1.2.@Transactional使用注意点

@Transactional注解只能在抛出RuntimeException或者Error时才会触发事务的回滚，常见的非RuntimeException是不会触发事务的回滚的。但是我们平时做业务处理时，需要捕获异常，所以可以手动抛出RuntimeException异常或者添加rollbackFor = Exception.class\(也可以指定相应异常\)

```
/*
 * 捕获异常时，要想使事务生效，需要手动抛出RuntimeException异常或者添加rollbackFor = Exception.class
*/
@Override
@Transactional
public Long addBook(Book book) {
    Long result = null;
    try {
        result = bookDao.addBook(book);
        int i = 1/0;
    } catch (Exception e) {
        e.printStackTrace();
        throw new RuntimeException();
    }
    return result;
}

@Override
@Transactional(rollbackFor = Exception.class)
public Long addBook(Book book) {
    Long result = null;
    try {
        result = bookDao.addBook(book);
        int i = 1/0;
    } catch (Exception e) {
        e.printStackTrace();
        throw e;
    }
    return result;
}
```

1.**只有public修饰的方法才会生效**

2.**方法内自调用导致的事务不生效**

**如下几种情况：**

```
/*
* 情况一：都有事务注解，异常在子方法出现，事务生效
*/
@Override
@Transactional
public Long addBook(Book book) {
    Long result = add(book);
    return result;
}

@Transactional
public Long add(Book book){
    Long result =  bookDao.addBook(book);
    int i = 1/0;
    return result;
}

/*
* 情况二：都有事务注解，异常在主方法出现，事务生效
*/
@Override
@Transactional
public Long addBook(Book book) {
    Long result = add(book);
    int i = 1/0;
    return result;
}

@Transactional
public Long add(Book book){
    Long result =  bookDao.addBook(book);
    return result;
}

/*
* 情况三：只有主方法有事务注解，异常在子方法出现，事务生效
*/
@Override
@Transactional
public Long addBook(Book book) {
    Long result = add(book);
    return result;
}

public Long add(Book book){
    Long result =  bookDao.addBook(book);
    int i = 1/0;
    return result;
}

/*
* 情况四：只有主方法有事务注解，异常在主方法出现，事务生效
*/
@Override
@Transactional
public Long addBook(Book book) {
    Long result = add(book);
    int i = 1/0;
    return result;
}

public Long add(Book book){
    Long result =  bookDao.addBook(book);
    return result;
}

/*
* 情况五：只有子方法有事务注解，异常在子方法出现，事务不生效
*/
@Override
public Long addBook(Book book) {
    Long result = add(book);
    return result;
}

@Transactional
public Long add(Book book){
    Long result =  bookDao.addBook(book);
    int i = 1/0;
    return result;
}
```

## 1.3.事务不生效

**结论：**

**当无事务方法调用有事务的方法时事务不会生效，**

**而主方法有事务去调用其他方法，无论被调用的方法有无事务，且是否出现异常（有异常需要能够抛出不被捕获），都触发事务。**

**如果你加了 try catch语句且不抛出异常，就相当于把异常吞了，这样当然没法触发事务，所以事务不会回滚；**

**数据库引擎要支持事务，如果是mysql，注意表要使用支持事务的引擎，比如innodb，如果是myisam，事务是不起作用的。**

**检查你的方法是不是public的。@Transactional注解只能应用到public可见度的方法上，如果应用在protected、private或者package可见度的方法上，也不会报错，不过事务设置不会起作用。**

## 1.4.@EnableTransactionManagement开启事务

在启动类上加上@EnableTransactionManagement// 开启注解事务管理，等同于xml配置文件中的&lt;tx:annotation-driven /&gt;

```
// 开启注解事务管理
@EnableTransactionManagement
@SpringBootApplication(scanBasePackages = {"com.gdfl.*"})
public class InformationServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(InformationServiceApplication.class, args);
    }

}
```

# 2.参考

一口气说出 6 种 @Transactional 注解的失效场景

[https://zhuanlan.zhihu.com/p/145897825](https://zhuanlan.zhihu.com/p/145897825)

@Transactional的使用：

[https://blog.csdn.net/u013929527/article/details/102596243](https://blog.csdn.net/u013929527/article/details/102596243)

[@Transactional 注解不生效原因](https://www.cnblogs.com/codingmengmeng/p/12111392.html)

[https://www.cnblogs.com/codingmengmeng/p/12111392.html](https://www.cnblogs.com/codingmengmeng/p/12111392.html)


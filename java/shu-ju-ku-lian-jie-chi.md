# 1.基本介绍

我们所熟知的C3P0，DBCP,Druid, HiKariCP为我们所常用的数据库连接池，

其中C3P0已经很久没有更新了。DBCP更新速度很慢，基本处于不活跃状态，而Druid和HikariCP处于活跃状态的更新中，这就是我们说的二代产品了。

20190523193131986.png

## 1.1.HiKariCP

字节码精简 ：优化代码，直到编译后的字节码最少，这样，CPU缓存可以加载更多的程序代码；

优化代理和拦截器 ：减少代码，例如HikariCP的Statement proxy只有100行代码，只有BoneCP的十分之一；

自定义数组类型（FastStatementList）代替ArrayList ：避免每次get\(\)调用都要进行range check，避免调用remove\(\)时的从头到尾的扫描；

自定义集合类型（ConcurrentBag ：提高并发读写的效率；

其他针对BoneCP缺陷的优化。

```
spring:
  datasource:
    name: ai
    url: ${MYSQL.URL:jdbc:mysql://192.168.1.28:3306/gdfl_ai?allowMultiQueries=true&useSSL=false}
    username: ${MYSQL.USERNAME:root}
    password: ${MYSQL.PASSWORD:Gdfl@123456}
    # 使用hikari数据源
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.jdbc.Driver
    hikari:
      max-lifetime: 120000 #一个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒以上
      maximum-pool-size: 25 #连接池中允许的最大连接数。缺省值：10；推荐的公式：((core_count * 2) + effective_spindle_count)
```

## 参考

[https://blog.csdn.net/qq\_17085463/article/details/90486515](https://blog.csdn.net/qq_17085463/article/details/90486515)


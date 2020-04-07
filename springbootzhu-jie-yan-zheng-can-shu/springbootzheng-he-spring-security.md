# 1.基本介绍

Spring Security 是 Spring 社区的一个顶级项目，也是 Spring Boot 官方推荐使用的安全框架。除了常规的认证（Authentication）和授权（Authorization）之外，**Spring Security还提供了诸如ACLs，LDAP，JAAS，CAS等高级特性以满足复杂场景下的安全需求**。另外，就目前而言，Spring Security和Shiro也是当前广大应用使用比较广泛的两个安全框架。

Spring Security 应用级别的安全主要包含两个主要部分，即**登录认证（Authentication）和访问授权（Authorization）**，首先用户登录的时候传入登录信息，登录验证器完成登录认证并将登录认证好的信息存储到请求上下文，然后再进行其他操作，如在进行接口访问、方法调用时，权限认证器从上下文中获取登录认证信息，然后根据认证信息获取权限信息，通过权限信息和特定的授权策略决定是否授权。

**Spring Security 另外一个强大之处就是它可以结合 OAuth2,玩出更多的花样出来。**

20190116102342618.jpg

20190813175708861.jpg

## 其主要过滤器

# 怎么使用

# 参考

[https://blog.csdn.net/qq\_22172133/article/details/86503223](https://blog.csdn.net/qq_22172133/article/details/86503223)


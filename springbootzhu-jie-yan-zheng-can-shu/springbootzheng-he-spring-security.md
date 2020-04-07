# 1.基本介绍

* 认证 （你是谁）
* 授权 （你能干什么）
* 攻击防护 （防止伪造身份）

Spring Security 是 Spring 社区的一个顶级项目，也是 Spring Boot 官方推荐使用的安全框架。除了常规的认证（Authentication）和授权（Authorization）之外，**Spring Security还提供了诸如ACLs，LDAP，JAAS，CAS等高级特性以满足复杂场景下的安全需求**。另外，就目前而言，Spring Security和Shiro也是当前广大应用使用比较广泛的两个安全框架。

Spring Security 应用级别的安全主要包含两个主要部分，即**登录认证（Authentication）和访问授权（Authorization）**，首先用户登录的时候传入登录信息，登录验证器完成登录认证并将登录认证好的信息存储到请求上下文，然后再进行其他操作，如在进行接口访问、方法调用时，权限认证器从上下文中获取登录认证信息，然后根据认证信息获取权限信息，通过权限信息和特定的授权策略决定是否授权。

**Spring Security 另外一个强大之处就是它可以结合 OAuth2,玩出更多的花样出来。**

**注意：绿色的过滤器可以配置是否生效，其他的都不能通过配置**

![img](/static/image/20190116102342618.jpg)

![img](/static/image/20190813175708861.jpg)

## 1.1.其主要过滤器

* WebAsyncManagerIntegrationFilter 
* SecurityContextPersistenceFilter 
* HeaderWriterFilter 
* CorsFilter 
* LogoutFilter
* RequestCacheAwareFilter
* SecurityContextHolderAwareRequestFilter
* AnonymousAuthenticationFilter
* SessionManagementFilter
* ExceptionTranslationFilter
* FilterSecurityInterceptor
* UsernamePasswordAuthenticationFilter
* BasicAuthenticationFilter

## 1.2.框架的核心组件

* SecurityContextHolder：提供对SecurityContext的访问
* SecurityContext,：持有Authentication对象和其他可能需要的信息
* AuthenticationManager 其中可以包含多个AuthenticationProvider
* ProviderManager对象为AuthenticationManager接口的实现类
* AuthenticationProvider 主要用来进行认证操作的类 调用其中的authenticate\(\)方法去进行认证操作
* Authentication：Spring Security方式的认证主体
* GrantedAuthority：对认证主题的应用层面的授权，含当前用户的权限信息，通常使用角色表示
* UserDetails：构建Authentication对象必须的信息，可以自定义，可能需要访问DB得到
* UserDetailsService：通过username构建UserDetails对象，通过loadUserByUsername根据userName获取（UserDetail对象 （可以在这里基于自身业务进行自定义的实现  如通过数据库，xml,缓存获取等）   

## 1.3.参数详解
1、注解 @EnableWebSecurity
     在 Spring boot 应用中使用 Spring Security，用到了 @EnableWebSecurity注解，官方说明为，该注解和 @Configuration 注解一起使用, 注解 WebSecurityConfigurer 类型的类，或者利用@EnableWebSecurity 注解继承 WebSecurityConfigurerAdapter的类，这样就构成了 Spring Security 的配置。

2、抽象类 WebSecurityConfigurerAdapter
     一般情况，会选择继承 WebSecurityConfigurerAdapter 类，其官方说明为：WebSecurityConfigurerAdapter 提供了一种便利的方式去创建 WebSecurityConfigurer的实例，只需要重写 WebSecurityConfigurerAdapter 的方法，即可配置拦截什么URL、设置什么权限等安全控制。

3、方法 configure(AuthenticationManagerBuilder auth) 和 configure(HttpSecurity http)
     Demo 中重写了 WebSecurityConfigurerAdapter 的两个方法：



## 1.4.JWT认证的实现
* 支持用户通过用户名和密码登录
* 登录后通过http header返回token，每次请求，客户端需通过header将token带回，用于权限校验
* 服务端负责token的定期刷新

# 2.怎么使用

# 3.参考

[https://blog.csdn.net/qq\_22172133/article/details/86503223](https://blog.csdn.net/qq_22172133/article/details/86503223)


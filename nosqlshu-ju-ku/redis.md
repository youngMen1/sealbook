# 1.Redis常用操作(管道(pipeline)实现批量操作,Redis模糊匹配等

## 1.1.开发环境准备
spring boot 2.x 使用RedisTemplate 操作,springboot项目pom引入redis依赖：

```
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency> 
        <!-- Redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-redis</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
        
```
Jedis是Redis官方推荐的面向Java的操作Redis的客户端，而RedisTemplate是SpringDataRedis中对JedisApi的高度封装。
SpringDataRedis相对于Jedis来说可以方便地更换Redis的Java客户端，比Jedis多了自动管理连接池的特性，方便与其他Spring框架进行搭配使用。

### redisTemplate
redisTemplate默认使用的是JDK序列化，但是可以主动设置

redisTemplate执行两条命令其实是在两个连接里完成的，因为redisTemplate执行完一个命令就会对其关闭，但是、

redisTemplate额外为什么提供了RedisCallback和SessionCallBack两个接口

### StringRedisTemplate
StringRedisTemplate继承RedisTemplate，只是提供字符串的操作，复杂的Java对象还要自行处理

### RedisCallback和SessionCallBack
作用: 让RedisTemplate进行回调，通过他们可以在同一条连接中执行多个redis命令
SessionCalback提供了良好的封装，优先使用它，
redisCallback使用起来有点复杂（很多工作需要我们自己来完成）还是优先选择SessionCalback


## 1.2.redis基础操作
### 1.2.1.redisTemplate模糊匹配删除

```
String key = "userRole:*";
redisTemplate.delete(key);
```
### 1.2.2.Redis模糊查询
可以通过Redis中keys命令进行获取key值，具体命令格式：keys pattern

文中提到redis中允许模糊查询的有3个通配符，分别是：*，?，[]

其中：
**使用通配符拿到keys**


```
Set<String> keysUserRole = redisTemplate.keys("userRole:" + "*");
```



## 1.3.Redis管道(pipeline)流操作

# 参考 
redis常用操作(管道(pipeline)实现批量操作,Redis模糊匹配等:`http://www.xiaoheidiannao.com/9747.html`
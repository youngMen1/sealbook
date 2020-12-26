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


## 1.2.redisTemplate模糊匹配
## 1.3.Redis管道(pipeline)流操作

# 参考 
redis常用操作(管道(pipeline)实现批量操作,Redis模糊匹配等:`http://www.xiaoheidiannao.com/9747.html`
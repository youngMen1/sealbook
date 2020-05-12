# 1.基本信息

# 如何使用RedisTemplate访问Redis数据结构

## 1.1.Redis 数据结构简介

Redis 可以存储键与5种不同数据结构类型之间的映射，这5种数据结构类型分别为String（字符串）、List（列表）、Set（集合）、Hash（散列）和 Zset（有序集合）。

下面来对这5种数据结构类型作简单的介绍：

| 结构类型 | 结构存储的值 | 结构的读写能力 |
| :--- | :--- | :--- |
| String | 可以是字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作；对象和浮点数执行自增\(increment\)或者自减\(decrement\) |
| List | 一个链表，链表上的每个节点都包含了一个字符串 | 从链表的两端推入或者弹出元素；根据偏移量对链表进行修剪\(trim\)；读取单个或者多个元素；根据值来查找或者移除元素 |
| Set | 包含字符串的无序收集器\(unorderedcollection\)，并且被包含的每个字符串都是独一无二的、各不相同 | 添加、获取、移除单个元素；检查一个元素是否存在于某个集合中；计算交集、并集、差集；从集合里卖弄随机获取元素 |
| Hash | 包含键值对的无序散列表 | 添加、获取、移除单个键值对；获取所有键值对 |
| Zset | 字符串成员\(member\)与浮点数分值\(score\)之间的有序映射，元素的排列顺序由分值的大小决定 | 添加、获取、删除单个元素；根据分值范围\(range\)或者成员来获取元素 |

Redis 5种数据结构的概念大致介绍到这边，下面将结合Spring封装的RedisTemplate来对这5种数据结构的运用进行演示

## RedisTemplate介绍

spring 封装了 RedisTemplate 对象来进行对redis的各种操作，它支持所有的 redis 原生的 api。

**RedisTemplate在spring代码中的结构如下：**

```
org.springframework.data.redis.core
Class RedisTemplate<K,V>
java.lang.Object
    org.springframework.data.redis.core.RedisAccessor
        org.springframework.data.redis.core.RedisTemplate<K,V>
```

Type Parameters:  
 K

* the Redis key type against which the template works \(usually a String\)

  模板中的Redis key的类型（通常为String）如：RedisTemplate&lt;String, Object&gt;

  注意：**如果没特殊情况，切勿定义成RedisTemplate&lt;Object, Object&gt;**  
  ，否则根据里氏替换原则，使用的时候会造成类型错误 。

  V

* the Redis value type against which the template works

  模板中的Redis value的类型

### RedisTemplate中定义了对5种数据结构操作

```
redisTemplate.opsForValue();//操作字符串
redisTemplate.opsForHash();//操作hash
redisTemplate.opsForList();//操作list
redisTemplate.opsForSet();//操作set
redisTemplate.opsForZSet();//操作有序set
```

## StringRedisTemplate与RedisTemplate

# 

# 3.参考

[https://www.jianshu.com/p/7bf5dc61ca06](https://www.jianshu.com/p/7bf5dc61ca06)


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

## 1.2.RedisTemplate介绍

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

## 1.3.StringRedisTemplate与RedisTemplate

两者的关系是StringRedisTemplate继承RedisTemplate。

* 两者的数据是不共通的；也就是说StringRedisTemplate只能管理StringRedisTemplate里面的数据，RedisTemplate只能管理RedisTemplate中的数据。

* SDR默认采用的序列化策略有两种，一种是String的序列化策略，一种是JDK的序列化策略。

  StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的。

  RedisTemplate默认采用的是JDK的序列化策略，保存的key和value都是采用此策略序列化保存的。

**RedisTemplate配置如下：**

```
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
{
    Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
    ObjectMapper om = new ObjectMapper();
    om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
    om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
    jackson2JsonRedisSerializer.setObjectMapper(om);
    RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
    template.setConnectionFactory(redisConnectionFactory);
    template.setKeySerializer(jackson2JsonRedisSerializer);
    template.setValueSerializer(jackson2JsonRedisSerializer);
    template.setHashKeySerializer(jackson2JsonRedisSerializer);
    template.setHashValueSerializer(jackson2JsonRedisSerializer);
    template.afterPropertiesSet();
    return template;
}
```

# 2.怎么使用

## 2.1.**Redis的String数据结构 （推荐使用StringRedisTemplate）**

**注意：如果使用RedisTemplate需要更改序列化方式：**

```
RedisSerializer<String> stringSerializer = new StringRedisSerializer();
        template.setKeySerializer(stringSerializer );
        template.setValueSerializer(stringSerializer );
        template.setHashKeySerializer(stringSerializer );
        template.setHashValueSerializer(stringSerializer );
```

public interface ValueOperations&lt;K,V&gt;  
 Redis operations for simple \(or in Redis terminology 'string'\) values.  
 ValueOperations可以对String数据结构进行操作：

* set void set\(K key, V value\);

```
使用：redisTemplate.opsForValue().set("name","tom");
结果：redisTemplate.opsForValue().get("name")  输出结果为tom
```

set void set\(K key, V value, long timeout, TimeUnit unit\);

```
使用：redisTemplate.opsForValue().set("name","tom",10, TimeUnit.SECONDS);
结果：redisTemplate.opsForValue().get("name")由于设置的是10秒失效，十秒之内查询有结果，十秒之后返回为null
```

* set void set\(K key, V value, long offset\);

  该方法是用 value 参数覆写\(overwrite\)给定 key 所储存的字符串值，从偏移量 offset 开始

```
使用：template.opsForValue().set("key","hello world");
        template.opsForValue().set("key","redis", 6);
        System.out.println("***************"+template.opsForValue().get("key"));
结果：***************hello redis
```

* setIfAbsent Boolean setIfAbsent\(K key, V value\);

```
使用：System.out.println(template.opsForValue().setIfAbsent("multi1","multi1"));//false  multi1之前已经存在
        System.out.println(template.opsForValue().setIfAbsent("multi111","multi111"));//true  multi111之前不存在
结果：false
true
```

* multiSet void multiSet\(Map&lt;? extends K, ? extends V&gt; m\);

  为多个键分别设置它们的值

```
使用：Map<String,String> maps = new HashMap<String, String>();
        maps.put("multi1","multi1");
        maps.put("multi2","multi2");
        maps.put("multi3","multi3");
        template.opsForValue().multiSet(maps);
        List<String> keys = new ArrayList<String>();
        keys.add("multi1");
        keys.add("multi2");
        keys.add("multi3");
        System.out.println(template.opsForValue().multiGet(keys));
结果：[multi1, multi2, multi3]
```

* multiSetIfAbsent Boolean multiSetIfAbsent\(Map&lt;? extends K, ? extends V&gt;m\);

为多个键分别设置它们的值，如果存在则返回false，不存在返回true

```
使用：Map<String,String> maps = new HashMap<String, String>();
        maps.put("multi11","multi11");
        maps.put("multi22","multi22");
        maps.put("multi33","multi33");
        Map<String,String> maps2 = new HashMap<String, String>();
        maps2.put("multi1","multi1");
        maps2.put("multi2","multi2");
        maps2.put("multi3","multi3");
        System.out.println(template.opsForValue().multiSetIfAbsent(maps));
        System.out.println(template.opsForValue().multiSetIfAbsent(maps2));
结果：true
false
```

* get V get\(Object key\);

```
使用：template.opsForValue().set("key","hello world");
        System.out.println("***************"+template.opsForValue().get("key"));
结果：***************hello world
```

getAndSet V getAndSet\(K key, V value\);

```
使用：template.opsForValue().set("getSetTest","test");
        System.out.println(template.opsForValue().getAndSet("getSetTest","test2"));
结果：test
```

* multiGet List&lt;V&gt; multiGet\(Collection&lt;K&gt; keys\);
   为多个键分别取出它们的值

```
使用：Map<String,String> maps = new HashMap<String, String>();
        maps.put("multi1","multi1");
        maps.put("multi2","multi2");
        maps.put("multi3","multi3");
        template.opsForValue().multiSet(maps);
        List<String> keys = new ArrayList<String>();
        keys.add("multi1");
        keys.add("multi2");
        keys.add("multi3");
        System.out.println(template.opsForValue().multiGet(keys));
结果：[multi1, multi2, multi3]
```

* increment Long increment\(K key, long delta\);
   支持整数

```
使用：template.opsForValue().increment("increlong",1);
        System.out.println("***************"+template.opsForValue().get("increlong"));
结果：***************1
```

* increment Double increment\(K key, double delta\);
   也支持浮点数

```
使用：template.opsForValue().increment("increlong",1.2);
        System.out.println("***************"+template.opsForValue().get("increlong"));
结果：***************2.2
```

* append Integer append\(K key, String value\);

  如果key已经存在并且是一个字符串，则该命令将该值追加到字符串的末尾。如果键不存在，则它被创建并设置为空字符串，因此APPEND在这种特殊情况下将类似于SET。

```
使用：template.opsForValue().append("appendTest","Hello");
        System.out.println(template.opsForValue().get("appendTest"));
        template.opsForValue().append("appendTest","world");
        System.out.println(template.opsForValue().get("appendTest"));
结果：Hello
        Helloworld
```

* get String get\(K key, long start, long end\);

  截取key所对应的value字符串

```
使用：appendTest对应的value为Helloworld
System.out.println("*********"+template.opsForValue().get("appendTest",0,5));
结果：*********Hellow
使用：System.out.println("*********"+template.opsForValue().get("appendTest",0,-1));
结果：*********Helloworld
使用：System.out.println("*********"+template.opsForValue().get("appendTest",-3,-1));
结果：*********rld
```

* size Long size\(K key\);

  返回key所对应的value值得长度

```
使用：template.opsForValue().set("key","hello world");
    System.out.println("***************"+template.opsForValue().size("key"));
结果：***************11
```

* setBit Boolean setBit\(K key, long offset, boolean value\);

  对 key 所储存的字符串值，设置或清除指定偏移量上的位\(bit\)

  key键对应的值value对应的ascii码,在offset的位置\(从左向右数\)变为value

```
使用：template.opsForValue().set("bitTest","a");
        // 'a' 的ASCII码是 97。转换为二进制是：01100001
        // 'b' 的ASCII码是 98  转换为二进制是：01100010
        // 'c' 的ASCII码是 99  转换为二进制是：01100011
        //因为二进制只有0和1，在setbit中true为1，false为0，因此我要变为'b'的话第六位设置为1，第七位设置为0
        template.opsForValue().setBit("bitTest",6, true);
        template.opsForValue().setBit("bitTest",7, false);
        System.out.println(template.opsForValue().get("bitTest"));
结果：b
```

* getBit Boolean getBit\(K key, long offset\);

  获取键对应值的ascii码的在offset处位值

```
使用：System.out.println(template.opsForValue().getBit("bitTest",7));
结果：false
```

## 2.2.Redis的List数据结构

**这边我们把RedisTemplate序列化方式改回之前的**

```
Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
       template.setKeySerializer(jackson2JsonRedisSerializer);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashKeySerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
```

public interface ListOperations&lt;K,V&gt;  
 Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）  
 ListOperations专门操作list列表：

* List&lt;V&gt;range\(K key, long start, long end\);
   返回存储在键中的列表的指定元素。偏移开始和停止是基于零的索引，其中0是列表的第一个元素（列表的头部），1是下一个元素

```
使用：System.out.println(template.opsForList().range("list",0,-1));
结果:[c#, c++, python, java, c#, c#]
```

* void trim\(K key, long start, long end\);

```
使用：System.out.println(template.opsForList().range("list",0,-1));
template.opsForList().trim("list",1,-1);//裁剪第一个元素
System.out.println(template.opsForList().range("list",0,-1));
结果:[c#, c++, python, java, c#, c#]
[c++, python, java, c#, c#]
```

* Long size\(K key\);

  返回存储在键中的列表的长度。如果键不存在，则将其解释为空列表，并返回0。当key存储的值不是列表时返回错误。

```
使用：System.out.println(template.opsForList().size("list"));
结果:6
```

* Long leftPush\(K key, V value\);

  将所有指定的值插入存储在键的列表的头部。如果键不存在，则在执行推送操作之前将其创建为空列表。（从左边插入）

```
使用：template.opsForList().leftPush("list","java");
        template.opsForList().leftPush("list","python");
        template.opsForList().leftPush("list","c++");
结果:返回的结果为推送操作后的列表的长度
1
2
3
```

* Long leftPushAll\(K key, V... values\);

  批量把一个数组插入到列表中

```
使用：String[] stringarrays = new String[]{"1","2","3"};
        template.opsForList().leftPushAll("listarray",stringarrays);
        System.out.println(template.opsForList().range("listarray",0,-1));
结果:[3, 2, 1]
```

* Long leftPushAll\(K key, Collection&lt;V&gt; values\);

  批量把一个集合插入到列表中

```
使用：List<Object> strings = new ArrayList<Object>();
        strings.add("1");
        strings.add("2");
        strings.add("3");
        template.opsForList().leftPushAll("listcollection4", strings);
        System.out.println(template.opsForList().range("listcollection4",0,-1));
结果:[3, 2, 1]
```

* Long leftPushIfPresent\(K key, V value\);

```

```

* Long leftPush\(K key, V pivot, V value\);
 
   把value值放到key对应列表中pivot值的左面，如果pivot值存在的话

```

```

* Long rightPush\(K key, V value\);
 
   将所有指定的值插入存储在键的列表的头部。如果键不存在，则在执行推送操作之前将其创建为空列表。（从右边插入）

```

```

* Long rightPushAll\(K key, V... values\);

```

```

* Long rightPushAll\(K key, Collection
  &lt;
  V
  &gt;
   values\);

```

```

* Long rightPushIfPresent\(K key, V value\);
 
   只有存在key对应的列表才能将这个value值插入到key所对应的列表中

```

```

* Long rightPush\(K key, V pivot, V value\);
 
   把value值放到key对应列表中pivot值的右面，如果pivot值存在的话

  


  




# 3.参考

[https://www.jianshu.com/p/7bf5dc61ca06](https://www.jianshu.com/p/7bf5dc61ca06)


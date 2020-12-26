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

### 1.1.1.redisTemplate
redisTemplate默认使用的是JDK序列化，但是可以主动设置

redisTemplate执行两条命令其实是在两个连接里完成的，因为redisTemplate执行完一个命令就会对其关闭，但是、

redisTemplate额外为什么提供了RedisCallback和SessionCallBack两个接口

### 1.1.2.StringRedisTemplate
StringRedisTemplate继承RedisTemplate，只是提供字符串的操作，复杂的Java对象还要自行处理

### 1.1.3.RedisCallback和SessionCallBack
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

**批量查询**

```
Set<String> keysList = stringRedisTemplate.keys(keys);
List<String> strings = stringRedisTemplate.opsForValue().multiGet(keysList);
```



## 1.3.Redis管道(pipeline)流操作
总的来说Redis的管道可以在大量数据需要一次性操作完成的时候,使用Pipeline进行批处理,将多次操作合并成一次操作,可以减少链路层的时间消耗。
流水线：
redis的读写速度十分快，所以系统的瓶颈往往是在网络通信中的延迟。
redis可能会在很多时候处于空闲状态而等待命令的到达。
为了解决这个问题，可以使用redis的流水线，流水线是一种通讯协议，类似一个队列批量执行一组命令。

### 1.3.1.redis的管道 pipeline批量set


```
@RequestMapping(value = "/redisPipeline", method = RequestMethod.POST)
    @ApiOperation(value = "redis的管道 pipeline 添加数据测试")
    public void  redistest(){
        log.info("redistest开始");
        // 开始时间
        long start = System.currentTimeMillis();
        RedisSerializer stringSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(stringSerializer);
        redisTemplate.setValueSerializer(stringSerializer);
        List<String> result = redisTemplate.executePipelined(new SessionCallback() {
            //执行流水线
            @Override
            public Object execute(RedisOperations operations) throws DataAccessException {
                //批量处理的内容
                for (int i = 0; i < 10000; i++) {
                    operations.opsForValue().set("redistest:" + "k" + i, "v" + i);
                }
                //注意这里一定要返回null，最终pipeline的执行结果，才会返回给最外层
                return null;
            }
        });
        // 结束时间
        long end = System.currentTimeMillis();
        log.info("运行时间："+(end-start));
        }
```
此处与未使用管道流水线操作做对比后续其他操作就不一一对比了
未使用流水线处理10000次请求：//耗时：5692；


```
public class RedisTest {
    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void test(){
        // 开始时间
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            redisTemplate.opsForValue().set("k"+i,"v"+i);
        }
        // 结束时间
        long end = System.currentTimeMillis();
        System.out.println(end-start);
    }
}
```
### 1.3.2.redis的管道 pipeline批量 delete

```
    @RequestMapping(value = "/redisPipeline", method = RequestMethod.DELETE)
    @ApiOperation(value = "redis的管道 pipeline删除测试")
    public void  redisDeletetest(){
        log.info("redistest开始");
        // 开始时间
        long start = System.currentTimeMillis();
        RedisSerializer stringSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(stringSerializer);
        redisTemplate.setValueSerializer(stringSerializer);
        redisTemplate.executePipelined(new SessionCallback() {
            //执行流水线
            @Override
            public Object execute(RedisOperations operations) throws DataAccessException {
                //批量处理的内容
                for (int i = 0; i < 20000; i++) {
                    //operations.opsForValue().set("redistest:"+"k"+i,"v"+i);
                    operations.delete("redistest:"+"k"+i);
                    System.out.println(i);
                }
                return null;
            }
        });
        // 结束时间
        long end = System.currentTimeMillis();
        log.info("运行时间："+(end-start));
    }
```

### 1.3.3.redis的管道 pipeline批量 GET


```
 /**
     * redis 批量操作其中一种方式
     * redis pipeline 管道技术
     */
    @RequestMapping(value = "/redisPipeline", method = RequestMethod.GET)
    @ApiOperation(value = "redis的管道 pipeline GET测试")
    public void redisPipeline(){
        RedisSerializer stringSerializer = new StringRedisSerializer();
        stringRedisTemplate.setKeySerializer(stringSerializer);
        stringRedisTemplate.setValueSerializer(stringSerializer);
        List<String>  keys=new ArrayList();
        for (int i = 0; i < 200; i++) {
            keys.add("redistest:"+"k"+i);
        }
        //调用 通道批量获取
        Map<String, Object> stringObjectMap = batchQueryByKeys(keys, true);
        System.out.println(stringObjectMap.size());
    }

    /**
     *
     * @param keys
     * @param useParallel  是否使用并行平行流
     * @return
     */
    public Map<String,Object> batchQueryByKeys(List<String> keys,Boolean useParallel){
        if(null == keys || keys.size() == 0 ){
            return null;
        }
        if(null == useParallel){
            useParallel = true;
        }
        List<Object> results = stringRedisTemplate.executePipelined(
                new RedisCallback<Object>() {
                    @Override
                    public Object doInRedis(RedisConnection connection) throws DataAccessException {
                        StringRedisConnection stringRedisConn = (StringRedisConnection)connection;
                        for(String key:keys) {
                            stringRedisConn.get(key);
                        }
                        return null;
                    }
                });
        if(null == results || results.size() == 0 ){return null;}

        Map<String,Object> resultMap  =  null;

        if(useParallel){

            Map<String,Object> resultMapOne  = Collections.synchronizedMap(new HashMap<String,Object>());

            keys.parallelStream().forEach(t -> {
                resultMapOne.put(t,results.get(keys.indexOf(t)));
            });

            resultMap = resultMapOne;

        }else{

            Map<String,Object> resultMapTwo  = new HashMap<>();

            for(String t:keys){
                resultMapTwo.put(t,results.get(keys.indexOf(t)));
            }

            resultMap = resultMapTwo;
        }

        return  resultMap;

    }
```
在这里要说明下我实现的管道pipeline批量获取使用的是RedisCallback对象实现的,原因是我使用SessionCalback对象来实现时调用get方法总是获取null最后也没找到原因所以使用了RedisCallback对象来实现的批量获取，如果有哪位大神了解SessionCalback对象的实现方法求指点一二

### 1.3.4.批量操作multi和pipeline效率的比较

multi和pipeline的区别在于multi会将操作都即刻的发送至redis服务端queued起来，每条指令queued的操作都有一次通信开销，执行exec时redis服务端再一口气执行queued队列里的指令，pipeline则是在客户端本地queued起来，执行exec时一次性的发送给redis服务端，这样只有一次通信开销。比如我有5个incr操作，multi的话这5个incr会有5次通信开销，但pipeline只有一次。

所以在批量操作使用pipeline效率会更高。

# 参考 
redis常用操作(管道(pipeline)实现批量操作,Redis模糊匹配等:`http://www.xiaoheidiannao.com/9747.html`
# 1.@Transactional注解的失效场景

@Transactional 注解相信大家并不陌生，平时开发中很常用的一个注解，它能保证方法内多个数据库操作要么同时成功、要么同时失败。使用@Transactional注解时需要注意许多的细节，不然你会发现@Transactional总是莫名其妙的就失效了。

## 1.1.事务

事务管理在系统开发中是不可缺少的一部分，Spring提供了很好事务管理机制，主要分为编程式事务和声明式事务两种。

**编程式事务：**是指在代码中手动的管理事务的提交、回滚等操作，代码侵入性比较强，如下示例：



```
try {
    //TODO something
     transactionManager.commit(status);
} catch (Exception e) {
    transactionManager.rollback(status);
    throw new InvoiceApplyException("异常失败");
}
```

**声明式事务：**基于AOP面向切面的，它将具体业务与事务处理部分解耦，代码侵入性很低，所以在实际开发中声明式事务用的比较多。声明式事务也有两种实现方式，一是基于TX和AOP的xml配置文件方式，二种就是基于@Transactional注解了。



```
@Transactional
    @GetMapping("/test")
    public String test() {
    
        int insert = cityInfoDictMapper.insert(cityInfoDict);
    }
```



## 1.2.@Transactional介绍

### 1.2.1.@Transactional注解可以作用于哪些地方？

@Transactional 可以作用在接口、类、类方法。

**作用于类：**当把@Transactional 注解放在类上时，表示所有该类的public方法都配置相同的事务属性信息。
**作用于方法：**当类配置了@Transactional，方法也配置了@Transactional，方法的事务会覆盖类的事务配置信息。
**作用于接口：**不推荐这种使用方法，因为一旦标注在Interface上并且配置了Spring AOP 使用CGLib动态代理，将会导致@Transactional注解失效



```
@Transactional
@RestController
@RequestMapping
public class MybatisPlusController {
    @Autowired
    private CityInfoDictMapper cityInfoDictMapper;
    
    @Transactional(rollbackFor = Exception.class)
    @GetMapping("/test")
    public String test() throws Exception {
        CityInfoDict cityInfoDict = new CityInfoDict();
        cityInfoDict.setParentCityId(2);
        cityInfoDict.setCityName("2");
        cityInfoDict.setCityLevel("2");
        cityInfoDict.setCityCode("2");
        int insert = cityInfoDictMapper.insert(cityInfoDict);
        return insert + "";
    }
}
```

### 1.2.2.@Transactional注有哪些属性？

#### propagation属性

propagation 代表事务的传播行为，默认值为 Propagation.REQUIRED，其他的属性信息如下：

* Propagation.REQUIRED：如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。( 也就是说如果A方法和B方法都添加了注解，在默认传播模式下，A方法内部调用B方法，会把两个方法的事务合并为一个事务 ）
* Propagation.SUPPORTS：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
* Propagation.MANDATORY：如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。
* Propagation.REQUIRES_NEW：重新创建一个新的事务，如果当前存在事务，暂停当前的事务。( 当类A中的 a 方法用默认Propagation.REQUIRED模式，类B中的 b方法加上采用 Propagation.REQUIRES_NEW模式，然后在 a 方法中调用 b方法操作数据库，然而 a方法抛出异常后，b方法并没有进行回滚，因为Propagation.REQUIRES_NEW会暂停 a方法的事务 )
* Propagation.NOT_SUPPORTED：以非事务的方式运行，如果当前存在事务，暂停当前的事务。
* Propagation.NEVER：以非事务的方式运行，如果当前存在事务，则抛出异常。
* Propagation.NESTED ：和 Propagation.REQUIRED 效果一样。

#### isolation 属性

isolation ：事务的隔离级别，默认值为 Isolation.DEFAULT。

* Isolation.DEFAULT：使用底层数据库默认的隔离级别。
* Isolation.READ_UNCOMMITTED
* Isolation.READ_COMMITTED
* Isolation.REPEATABLE_READ
* Isolation.SERIALIZABLE

#### timeout 属性

timeout ：事务的超时时间，默认值为 -1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

#### readOnly 属性

readOnly ：指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。

#### rollbackFor 属性

rollbackFor ：用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。

#### noRollbackFor属性**
noRollbackFor：抛出指定的异常类型，不回滚事务，也可以指定多个异常类型。

# 2.总结

# 3.参考
一口气说出 6种，@Transactional注解的失效场景
https://zhuanlan.zhihu.com/p/114461128

# 1.@Transactional注解的失效场景

@Transactional 注解相信大家并不陌生，平时开发中很常用的一个注解，它能保证方法内多个数据库操作要么同时成功、要么同时失败。使用@Transactional注解时需要注意许多的细节，不然你会发现@Transactional总是莫名其妙的就失效了。

## 事务

事务管理在系统开发中是不可缺少的一部分，Spring提供了很好事务管理机制，主要分为编程式事务和声明式事务两种。

编程式事务：是指在代码中手动的管理事务的提交、回滚等操作，代码侵入性比较强，如下示例：



```
try {
    //TODO something
     transactionManager.commit(status);
} catch (Exception e) {
    transactionManager.rollback(status);
    throw new InvoiceApplyException("异常失败");
}
```





# 2.总结

# 3.参考
一口气说出 6种，@Transactional注解的失效场景
https://zhuanlan.zhihu.com/p/114461128

## Java:如何更优雅的处理空值 {#activity-name}

导语

在笔者几年的开发经验中，经常看到项目中存在到处空值判断的情况，这些判断，会让人觉得摸不着头绪，它的出现很有可能和当前的业务逻辑并没有关系。但它会让你很头疼。

有时候，更可怕的是系统因为这些空值的情况，会抛出空指针异常，导致业务系统发生问题。

此篇文章，我总结了几种关于空值的处理手法，希望对读者有帮助。

## 业务中的空值

```
public List<User> listUser(){
    List<User> userList = userListRepostity.selectByExample(new UserExample());
    if(CollectionUtils.isEmpty(userList)){//spring util工具类
      return null;
    }
    return userList;
}
```

这段代码返回是null,从我多年的开发经验来讲，对于集合这样返回值，最好不要返回null，因为如果返回了null，会给调用者带来很多麻烦。你将会把这种调用风险交给调用者来控制。

如果调用者是一个谨慎的人，他会进行是否为null的条件判断。如果他并非谨慎，或者他是一个面向接口编程的狂热分子\(当然，面向接口编程是正确的方向\)，他会按照自己的理解去调用接口，而不进行是否为null的条件判断，如果这样的话，是非常危险的，它很有可能出现空指针异常！

根据墨菲定律来判断: **“很有可能出现的问题，在将来一定会出现!”**

基于此，我们将它进行优化:

```
public List<User> listUser(){
    List<User> userList = userListRepostity.selectByExample(new UserExample());
    if(CollectionUtils.isEmpty(userList)){
      return Lists.newArrayList();//guava类库提供的方式
    }
    return userList;

}
```

#### 使用Optional可以进行优化

## 参考:

[https://mp.weixin.qq.com/s/SX1puW0KSuCcwi4MI5jLGg](https://mp.weixin.qq.com/s/SX1puW0KSuCcwi4MI5jLGg)


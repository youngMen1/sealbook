# Java之Collections.emptyList()、emptySet()、emptyMap()的作用和好处以及要注意的地方
1，如果你想 new 一个空的 List ，而这个 List 以后也不会再添加元素（有大坑，看下面更新），

那么就用 Collections.emptyList() 好了。
new ArrayList() 或者 new LinkedList() 在创建的时候有会有初始大小，多少会占用一内存。
每次使用都new 一个空的list集合，浪费就积少成多，浪费就严重啦，就不好啦
2，为了编码的方便。
比如说一个方法返回类型是List，当没有任何结果的时候，返回null,有结果的时候，返回list集合列表。
那样的话，调用这个方法的地方，就需要进行null判断。使用emptyList这样的方法，可以方便方法调用者。返回的就不会是null，省去重复代码。

注意的地方：
这个空的集合是不能调用.add（），添加元素的。因为直接报异常。因为源码就是这么写的：直接抛异常。

哦，Collections里面没这么写，但是EmptyList继承了AbstractList这个抽象类，里面简单实现了部分集合框架的方法。
这里面的add方法最后调用的方法体，就是直接抛异常。
throw new UnsupportedOperationException();
这么解释add报异常就对啦。
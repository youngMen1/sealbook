## 概述

工具类 就是封装平常用的方法，不需要你重复造轮子，节省开发人员时间，提高工作效率。谷歌作为大公司，当然会从日常的工作中提取中很多高效率的方法出来。所以就诞生了guava。

guava的优点：

* 高效设计良好的API，被Google的开发者设计，实现和使用
* 遵循高效的java语法实践
* 使代码更刻度，简洁，简单
* 节约时间，资源，提高生产力

Guava工程包含了若干被Google的 Java项目广泛依赖 的核心库，例如：

* 集合 \[collections\]
* 缓存 \[caching\]
* 原生类型支持 \[primitives support\]
* 并发库 \[concurrency libraries\]
* 通用注解 \[common annotations\]
* 字符串处理 \[string processing\]
* I/O 等等。

## G**oogle Collections**

```
Map<String, Map<Long, List<String>>> map = new HashMap<String, Map<Long,List<String>>>();
```

可以这么写:

```
Map<String, Map<Long, List<String>>> map = Maps.newHashMap();
或者更甚者直接使用静态导入:
Map<String, Map<Long, List<String>>>map = newHashMap();
```

Lists和Sets也有：

```
Lists.newArrayList();
Sets.newHashSet();
```

**操作lists和maps:**

当你在写单元测试时，经常会构造一些测试数据，可能是list、map、set等，对于一些像我一样草率的人来说，测试代码中会经常看到类似下面的语句：

```
List<String> list = new ArrayList<String>();
list.add("a");
list.add("b");
list.add("c");
list.add("d");
```

其实我也知道，这几行代码看起来很烂，我只是想用一些测试数据构造一个不可变的list而已，我希望能像下面这样写一行代码搞定这些。。如何办到？好吧，这很简单！

```
ImmutableList<String> of = ImmutableList.of("a", "b", "c", "d");
```

Map也一样：

```
ImmutableList<String> list2 = listOf("a", "b", "c", "d");
```

而且如果我想构造填充一个ArrayList（或者一个HashMap），我可以这样：

```
ArrayList<String> list3 = arrayListOf("a", "b", "c", "d");
```

---

## 参考:

http://ifeve.com/google-guava/




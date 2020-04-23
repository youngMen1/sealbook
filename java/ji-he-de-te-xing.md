# 1.合理利用好集合的特性有助于事半功倍

## 1.1.Collection的集合类的继承树

![img](/static/image/20190717224652123.png)

### 1.1.1.List集合

它是一个元素存取有序的集合。例如，存元素的顺序是11、22、33。那么集合中，元素的存储就是按照11、22、33的顺序完成的），集合中可以有重复的元素。

#### 1.1.1.1.ArrayList

ArrayList集合数据存储的结构是数组结构。元素增删慢，查找快，由于日常开发中使用最多的功能为查询数据、遍历数据，所以ArrayList是最常用的集合，线程不安全。

许多程序员开发时非常随意地使用ArrayList完成任何需求，并不严谨，这种用法是不提倡的。

#### 1.1.1..2.LinkedList

LinkedList集合数据存储的结构是链表结构。方便元素添加、删除的集合。元素增删快，查找慢，实际开发中对一个集合元素的添加与删除经常涉及到首尾操作，而LinkedList提供了大量首尾操作的方法，线程不安全。

### 1.1.2.Set集合

* Set集合，基础自Collection。特征是插入无序，不可指定位置访问。

* Set集合的实现类可说是基于Map集合去写的。通过内部封装Map集合来实现的比如HashSet内部封装了HashMap。

* Set集合的数据库不能重复（== 或 eqauls）的元素

* Set集合的常用实现类有 HashSet TreeSet

#### 1.1.2.1.HashSet

内部无序，封装了HashMap。就是用HashMap的key位来存储值.

```
//set集合内部的封装的HashMap对象
private transient HashMap<E,Object> map;
```

```
Set<String> set = new HashSet<>();
        set.add("1");
        set.add("1");
        set.add("1");
        set.add("1");
        // set:[1]
        // size:1
        System.out.println("set:" + set.toString());
        System.out.println("size:" + set.size());
```

执行结果是 size:1.  
详细用过Map集合的你很明白这一点。  
如果添加的元素相==或equals HashSet就只会保留其中一个。  
当我们将自己写的类存入set集合时一定要重写 equals和hashCode

#### 1.1.2.2.TreeSet

TreeSet基于TreeMap实现，TreeMap本质就是红黑树。所以TreeSet其实于是基于红黑树的。

TreeSet有个特点，插入无序内部有序。

插入数据实现Comparable接口，通过compareTo方法去比较大小，或者在实力化TreeSet的时候自定义排序Comparator方法。内部的int compare\(T o1, T o2\)比较对象大小。

如果插入数据即不实现Commparable或在插入的时候也不指定排序方式Comparator那么就会报错

注：如果不了解Comparable和Comparator可用参考我写的这篇博客：[https://www.cnblogs.com/IT-CPC/p/10903837.html](https://www.cnblogs.com/IT-CPC/p/10903837.html)

**证明插入无序内部有序**

```
// 插入无序内部有序
        Set<String> treeSet = new TreeSet<>();
        treeSet.add("b");
        treeSet.add("a");
        treeSet.add("d");
        treeSet.add("c");

        for (String str : treeSet) {
            System.out.println("treeSet:" + str);
        }

treeSet:a
treeSet:b
treeSet:c
treeSet:d
```

如果我们自己实现的对象要实现排序 1、 实现Comparable 2、指定排序方式 Comparator

#### HashSet和TreeSet比较

HashSet 效率要高于TreeSet，因为HashSet采用散列算法快速对集合进行增删改查 时间复杂度更是几乎接近O\(1\)，但内部无序。

TreeSet 内部有序，可根据指定规则去排序。但效率要比较HashSet低。时间复杂度位O\(log n\)

forEach底层其实就是依赖迭代器。很多人喜欢用ForEach因为代码少。

```
for (Object object : set) {
            System.out.println(object);
        }

// jdk8新特性
set.forEach((x) -> System.out.println(x));
```

## 1.2.Map 类集合 K/V

Map的继承关系：

![img](/static/image/1685101-20190520015745840-1408257336.png)

![img](/static/image/微信截图_20200423170437.png)

### 1.2.1.HashMap

### 1.2.2.LinkedHashMap

LinkedHashMap是HashMap的子类，使用一个双向链表来维护键值对的顺序（与添加顺序相同），维护的是key的顺序。

存储、查找仍是使用hashCode值来索引，遍历（迭代）才使用链表，就是说遍历时的顺序和添加元素的顺序相同。

因为内部维护了一个链表，所以查找、存储时HashMap性能高于LinkedHashMap，但遍历集合时LinkedHashMap性能较好。

### 1.2.3.HashTable

### 1.2.4.TreeMap

TreeMap类是Map接口的子接口SortedMap的实现类，用红黑树来存储元素，保持集合的有序（并不是添加顺序，而是按key的值排列）。

TreeMap的2中排序方式：自然排序（默认）、定制排序。

TreeMap的键值对（其实是key）总是处于有序状态，调用keySet\(\)获取key的Set，再使用toArray\(\)转化为数组，这个数组就是有序的。

```

```

### 1.2.5.Properties

Properties是Hashtable的子类，在处理属性文件时特别方便。Properties的key、value都必须是String。

Properties类的常用方法：

String getProperty\(String key\)

String getProperty\(String key,String defaultValue\)   key不存在时，把默认的值作为value返回

Object setProperty\(String key,String value\)   设置属性值。相当于Hashtable的put\(\)

void  load\(InputStream is\)   从属性文件（输入流）中加载键值对到Properties对象

void store\(OutputStream os,String info\)   把properties中的键值对写到输出流中（一般是属性文件）。第二个参数是对properties信息的说明。

```
Properties properties=new Properties();
      properties.setProperty("name","张三");
      //key、value都必须是String
      properties.setProperty("age","19");  
      //输出键值对到属性文件，一行一个键值对。第二个参数是对文件所保存信息的说明，会以注释形式输出到属性文件的首行。a.ini默认在项目的根目录下。
      properties.store(new FileOutputStream("a.ini"),"student info");
      //properties中仍有键值对
      System.out.println(properties);  //{name=张三, age=19}
```

### 1.2.6.EnumMap

EnumMap的key都必须是同一个枚举类的实例，元素的排列顺序与枚举类中的顺序保持一致。

## 参考

**Set集合详解：**

[https://www.cnblogs.com/IT-CPC/p/10904074.html](https://www.cnblogs.com/IT-CPC/p/10904074.html)


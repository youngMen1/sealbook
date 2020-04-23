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

#### 1.1.2.2.TreeSet

## 1.2.Map 类集合 K/V

Map的继承关系：

![img](/static/image/1685101-20190520015745840-1408257336.png)

### 1.2.1.HashMap

### 1.2.2.LinkedHashMap

### 1.2.3.HashTable

### 1.2.4.TreeMap




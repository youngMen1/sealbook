# Guava对JDK集合的扩展，这是Guava最成熟和为人所知的部分

## 用不变的集合进行防御性编程和性能提升。

## 2.1-不可变集合

## 范例

```
public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
        "red",
        "orange",
        "yellow",
        "green",
        "blue",
        "purple");

class Foo {
    Set<Bar> bars;
    Foo(Set<Bar> bars) {
        this.bars = ImmutableSet.copyOf(bars); // defensive copy!
    }
}
```

## 为什么要使用不可变集合

不可变对象有很多优点，包括：

* 当对象被不可信的库调用时，不可变形式是安全的；
* 不可变对象被多个线程调用时，不存在竞态条件问题
* 不可变集合不需要考虑变化，因此可以节省时间和空间。所有不可变的集合都比它们的可变形式有更好的内存利用率（分析和测试细节）；
* 不可变对象因为有固定不变，可以作为常量来安全使用。

创建对象的不可变拷贝是一项很好的防御性编程技巧。Guava为所有JDK标准集合类型和Guava新集合类型都提供了简单易用的不可变版本。  
 JDK也提供了Collections.unmodifiableXXX方法把集合包装为不可变形式，但我们认为不够好：

* 笨重而且累赘：不能舒适地用在所有想做防御性拷贝的场景；
* 不安全：要保证没人通过原集合的引用进行修改，返回的集合才是事实上不可变的；
* 低效：包装过的集合仍然保有可变集合的开销，比如并发修改的检查、散列表的额外空间，等等。

如果你没有修改某个集合的需求，或者希望某个集合保持不变时，把它防御性地拷贝到不可变集合是个很好的实践。

重要提示：所有Guava不可变集合的实现都不接受null值。我们对Google内部的代码库做过详细研究，发现只有5%的情况需要在集合中允许null元素，剩下的95%场景都是遇到null值就快速失败。如果你需要在不可变集合中使用null，请使用JDK中的Collections.unmodifiableXXX方法。更多细节建议请参考“使用和避免null”。

## 怎么使用不可变集合

不可变集合可以用如下多种方式创建：

* copyOf方法，如ImmutableSet.copyOf\(set\);
* of方法，如ImmutableSet.of\(“a”, “b”, “c”\)或 ImmutableMap.of\(“a”, 1, “b”, 2\);
* Builder工具，如

```
public static final ImmutableSet<Color> GOOGLE_COLORS =
        ImmutableSet.<Color>builder()
            .addAll(WEBSAFE_COLORS)
            .add(new Color(0, 191, 255))
            .build();
```

### 比想象中更智能的copyOf

请注意，ImmutableXXX.copyOf方法会尝试在安全的时候避免做拷贝——实际的实现细节不详，但通常来说是很智能的，比如：

```
ImmutableSet<String> foobar = ImmutableSet.of("foo", "bar", "baz");
thingamajig(foobar);

void thingamajig(Collection<String> collection) {
    ImmutableList<String> defensiveCopy = ImmutableList.copyOf(collection);
    ...
}
```

在这段代码中，ImmutableList.copyOf\(foobar\)会智能地直接返回foobar.asList\(\),它是一个ImmutableSet的常量时间复杂度的List视图。  
作为一种探索，ImmutableXXX.copyOf\(ImmutableCollection\)会试图对如下情况避免线性时间拷贝：

* 在常量时间内使用底层数据结构是可能的——例如，ImmutableSet.copyOf\(ImmutableList\)就不能在常量时间内完成。
* 不会造成内存泄露——例如，你有个很大的不可变集合ImmutableList&lt;String&gt;

  hugeList， ImmutableList.copyOf\(hugeList.subList\(0, 10\)\)就会显式地拷贝，以免不必要地持有hugeList的引用。

* 不改变语义——所以ImmutableSet.copyOf\(myImmutableSortedSet\)会显式地拷贝，因为和基于比较器的ImmutableSortedSet相比，ImmutableSet对hashCode\(\)和equals有不同语义。

在可能的情况下避免线性拷贝，可以最大限度地减少防御性编程风格所带来的性能开销。

### asList视图

所有不可变集合都有一个asList\(\)方法提供ImmutableList视图，来帮助你用列表形式方便地读取集合元素。例如，你可以使用sortedSet.asList\(\).get\(k\)从ImmutableSortedSet中读取第k个最小元素。

asList\(\)返回的ImmutableList通常是——并不总是——开销稳定的视图实现，而不是简单地把元素拷贝进List。也就是说，asList返回的列表视图通常比一般的列表平均性能更好，比如，在底层集合支持的情况下，它总是使用高效的contains方法。

## 细节：关联可变集合和不可变集合

| **可变集合接口** | **属于JDK还是Guava** | **不可变版本** |
| :--- | :--- | :--- |
| Collection | JDK | [ImmutableCollection](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableCollection.html) |
| List | JDK | [ImmutableList](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableList.html) |
| Set | JDK | [ImmutableSet](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableSet.html) |
| SortedSet/NavigableSet | JDK | [ImmutableSortedSet](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableSortedSet.html) |
| Map | JDK | [ImmutableMap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableMap.html) |
| SortedMap | JDK | [ImmutableSortedMap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableSortedMap.html) |
| [Multiset](http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Multiset) | Guava | [ImmutableMultiset](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableMultiset.html) |
| SortedMultiset | Guava | [ImmutableSortedMultiset](http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/ImmutableSortedMultiset.html) |
| [Multimap](http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Multimap) | Guava | [ImmutableMultimap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableMultimap.html) |
| ListMultimap | Guava | [ImmutableListMultimap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableListMultimap.html) |
| SetMultimap | Guava | [ImmutableSetMultimap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableSetMultimap.html) |
| [BiMap](http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#BiMap) | Guava | [ImmutableBiMap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableBiMap.html) |
| [ClassToInstanceMap](http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#ClassToInstanceMap) | Guava | [ImmutableClassToInstanceMap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableClassToInstanceMap.html) |
| [Table](http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Table) | Guava | [ImmutableTable](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableTable.html) |

---

## 2.3-强大的集合工具类：java.util.Collections中未包含的集合工具

任何对JDK集合框架有经验的程序员都熟悉和喜欢[java.util.Collections](http://docs.oracle.com/javase/7/docs/api/java/util/Collections.html)包含的工具方法。Guava沿着这些路线提供了更多的工具方法：适用于所有集合的静态方法。这是Guava最流行和成熟的部分之一。

我们用相对直观的方式把工具类与特定集合接口的对应关系归纳如下：

| **集合接口** | **属于JDK还是Guava** | **对应的Guava工具类** |
| :--- | :--- | :--- |
| Collection | JDK | [Collections2](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Collections2.html)：不要和java.util.Collections混淆 |
| List | JDK | [Lists](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Lists.html) |
| Set | JDK | [Sets](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Sets.html) |
| SortedSet | JDK | [Sets](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Sets.html) |
| Map | JDK | [Maps](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Maps.html) |
| SortedMap | JDK | [Maps](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Maps.html) |
| Queue | JDK | [Queues](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Queues.html) |
| [Multiset](http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Multiset) | Guava | [Multisets](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multisets.html) |
| [Multimap](http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Multimap) | Guava | [Multimaps](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html) |
| [BiMap](http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#BiMap) | Guava | [Maps](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html) |
| [Table](http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Table) | Guava | [Tables](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Tables.html) |

在找类似转化、过滤的方法？请看第四章，函数式风格。

在JDK 7之前，构造新的范型集合时要讨厌地重复声明范型：

```
List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<TypeThatsTooLongForItsOwnGood>();
```

我想我们都认为这很讨厌。因此Guava提供了能够推断范型的静态工厂方法：

```
List<TypeThatsTooLongForItsOwnGood> list = Lists.newArrayList();
Map<KeyType, LongishValueType> map = Maps.newLinkedHashMap();
```

可以肯定的是，JDK7版本的钻石操作符\(&lt;&gt;\)没有这样的麻烦：

```
List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<>();
```

但Guava的静态工厂方法远不止这么简单。用工厂方法模式，我们可以方便地在初始化时就指定起始元素。

```
Set<Type> copySet = Sets.newHashSet(elements);
List<String> theseElements = Lists.newArrayList("alpha", "beta", "gamma");
```




## 1.1-[使用和避免null](http://ifeve.com/using-and-avoiding-null/)

null是模棱两可的，会引起令人困惑的错误，有些时候它让人很不舒服。很多Guava工具类用快速失败拒绝null值，而不是盲目地接受。

## **Optional**

大多数情况下，开发人员使用null表明的是某种缺失情形：可能是已经有一个默认值，或没有值，或找不到值。例如，Map.get返回null就表示找不到给定键对应的值。

Guava用[Optional&lt;T&gt;](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html)表示可能为null的T类型引用。一个Optional实例可能包含非null的引用（我们称之为引用存在），也可能什么也不包括（称之为引用缺失）。它从不说包含的是null值，而是用存在或缺失来表示。但Optional从不会包含null值引用。

**创建Optional实例（以下都是静态方法）：**

| [Optional.of\(T\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#of%28T%29) | 创建指定引用的Optional实例，若引用为null则快速失败 |
| :--- | :--- |
| [Optional.absent\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#absent%28%29) | 创建引用缺失的Optional实例 |
| [Optional.fromNullable\(T\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#fromNullable%28T%29) | 创建指定引用的Optional实例，若引用为null则表示缺失 |

**用Optional实例查询引用（以下都是非静态方法）：**

| [boolean isPresent\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#isPresent%28%29) | 如果Optional包含非null的引用（引用存在），返回true |
| :--- | :--- |
| [T get\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#get%28%29) | 返回Optional所包含的引用，若引用缺失，则抛出java.lang.IllegalStateException |
| [T or\(T\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#or%28T%29) | 返回Optional所包含的引用，若引用缺失，返回指定的值 |
| [T orNull\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#orNull%28%29) | 返回Optional所包含的引用，若引用缺失，返回null |
| [Set&lt;T&gt; asSet\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#asSet%28%29) | 返回Optional所包含引用的单例不可变集，如果引用存在，返回一个只有单一元素的集合，如果引用缺失，返回一个空集合。 |

**使用Optional的意义在哪儿？**

使用Optional除了赋予null语义，增加了可读性，最大的优点在于它是一种傻瓜式的防护。Optional迫使你积极思考引用缺失的情况，因为你必须显式地从Optional获取引用。直接使用null很容易让人忘掉某些情形，尽管FindBugs可以帮助查找null相关的问题，但是我们还是认为它并不能准确地定位问题根源。

如同输入参数，方法的返回值也可能是null。和其他人一样，你绝对很可能会忘记别人写的方法method\(a,b\)会返回一个null，就好像当你实现method\(a,b\)时，也很可能忘记输入参数a可以为null。将方法的返回类型指定为Optional，也可以迫使调用者思考返回的引用缺失的情形。

---

## 1.2-前置条件

前置条件：让方法调用的前置条件判断更简单。

没有额外参数：抛出的异常中没有错误消息；

* 有一个Object对象作为额外参数：抛出的异常使用Object.toString\(\) 作为错误消息；
* 有一个String对象作为额外参数，并且有一组任意数量的附加Object对象：这个变种处理异常消息的方式有点类似printf，但考虑GWT的兼容性和效率，只支持%s指示符。

| **方法声明（不包括额外参数）** |
| :--- |


|  | **描述** | **检查失败时抛出的异常** |
| :--- | :--- | :--- |
| [checkArgument\(boolean\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkArgument%28boolean%29) | 检查boolean是否为true，用来检查传递给方法的参数。 | IllegalArgumentException |
| [checkNotNull\(T\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkNotNull%28T%29) | 检查value是否为null，该方法直接返回value，因此可以内嵌使用checkNotNull。 | NullPointerException |
| [checkState\(boolean\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkState%28boolean%29) | 用来检查对象的某些状态。 | IllegalStateException |
| [checkElementIndex\(int index, int size\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkElementIndex%28int, int%29) | 检查index作为索引值对某个列表、字符串或数组是否有效。index&gt;=0 && index&lt;size \* | IndexOutOfBoundsException |
| [checkPositionIndex\(int index, int size\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkPositionIndex%28int, int%29) | 检查index作为位置值对某个列表、字符串或数组是否有效。index&gt;=0 && index&lt;=size \* | IndexOutOfBoundsException |
| [checkPositionIndexes\(int start, int end, int size\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkPositionIndexes%28int, int, int%29) | 检查\[start, end\]表示的位置范围对某个列表、字符串或数组是否有效\* | IndexOutOfBoundsException |

\*索引值常用来查找列表、字符串或数组中的元素，如List.get\(int\), String.charAt\(int\)

_\*位置值和位置范围常用来截取列表、字符串或数组，如List.subList\(int，int\), String.substring\(int\)_

相比Apache Commons提供的类似方法，我们把Guava中的Preconditions作为首选。Piotr Jagielski在[他的博客](http://piotrjagielski.com/blog/google-guava-vs-apache-commons-for-argument-validation/)中简要地列举了一些理由：

* 在静态导入后，Guava方法非常清楚明晰。checkNotNull清楚地描述做了什么，会抛出什么异常；
* checkNotNull直接返回检查的参数，让你可以在构造函数中保持字段的单行赋值风格：this.field = checkNotNull\(field\)
* 简单的、参数可变的printf风格异常信息。鉴于这个优点，在JDK7已经引入
  [Objects.requireNonNull](http://docs.oracle.com/javase/7/docs/api/java/util/Objects.html#requireNonNull%28java.lang.Object,java.lang.String%29)
  的情况下，我们仍然建议你使用checkNotNull。

在编码时，如果某个值有多重的前置条件，我们建议你把它们放到不同的行，这样有助于在调试时定位。此外，把每个前置条件放到不同的行，也可以帮助你编写清晰和有用的错误消息。

---

## 1.4-排序: Guava强大的”流畅风格比较器”

[排序器\[Ordering\]](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html)是Guava流畅风格比较器\[Comparator\]的实现，它可以用来为构建复杂的比较器，以完成集合排序的功能。

从实现上说，Ordering实例就是一个特殊的Comparator实例。Ordering把很多基于Comparator的静态方法（如Collections.max）包装为自己的实例方法（非静态方法），并且提供了链式调用方法，来定制和增强现有的比较器。

**创建排序器**：常见的排序器可以由下面的静态方法创建

| **方法** | **描述** |
| :--- | :--- |
| [natural\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#natural%28%29) | 对可排序类型做自然排序，如数字按大小，日期按先后排序 |
| [usingToString\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#usingToString%28%29) | 按对象的字符串形式做字典排序\[lexicographical ordering\] |
| [from\(Comparator\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#from%28java.util.Comparator%29) | 把给定的Comparator转化为排序器 |

```
Ordering<String> byLengthOrdering = new Ordering<String>() {
  public int compare(String left, String right) {
    return Ints.compare(left.length(), right.length());
  }
};
```

**链式调用方法**：通过链式调用，可以由给定的排序器衍生出其它排序器

| **方法** | **描述** |
| :--- | :--- |
| [reverse\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#reverse%28%29) | 获取语义相反的排序器 |
| [nullsFirst\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#nullsFirst%28%29) | 使用当前排序器，但额外把null值排到最前面。 |
| [nullsLast\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#nullsLast%28%29) | 使用当前排序器，但额外把null值排到最后面。 |
| [compound\(Comparator\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#compound%28java.util.Comparator%29) | 合成另一个比较器，以处理当前排序器中的相等情况。 |
| [lexicographical\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#lexicographical%28%29) | 基于处理类型T的排序器，返回该类型的可迭代对象Iterable&lt;T&gt;的排序器。 |
| [onResultOf\(Function\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#onResultOf%28com.google.common.base.Function%29) | 对集合中元素调用Function，再按返回值用当前排序器排序。 |

```
class Foo {
    @Nullable 
    String sortedBy;
    int notSortedBy;
}
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(new Function<Foo, String>() {
  public String apply(Foo foo) {
    return foo.sortedBy;
  }
});
```

注：用compound方法包装排序器时，就不应遵循从后往前读的原则。为了避免理解上的混乱，请不要把compound写在一长串链式调用的中间，你可以另起一行，在链中最先或最后调用compound。

超过一定长度的链式调用，也可能会带来阅读和理解上的难度。我们建议按下面的代码这样，在一个链中最多使用三个方法。此外，你也可以把Function分离成中间对象，让链式调用更简洁紧凑

```
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(sortKeyFunction)
```

**运用排序器：**

Guava的排序器实现有若干操纵集合或元素值的方法

| **方法** | **描述** | **另请参见** |
| :--- | :--- | :--- |
| [greatestOf\(Iterable iterable, int k\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#greatestOf%28java.lang.Iterable, int%29) | 获取可迭代对象中最大的k个元素。 | [leastOf](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#leastOf%28java.lang.Iterable, int%29) |
| [isOrdered\(Iterable\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#isOrdered%28java.lang.Iterable%29) | 判断可迭代对象是否已按排序器排序：允许有排序值相等的元素。 | [isStrictlyOrdered](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#isStrictlyOrdered%28java.lang.Iterable%29) |
| [sortedCopy\(Iterable\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#sortedCopy%28java.lang.Iterable%29) | 判断可迭代对象是否已严格按排序器排序：不允许排序值相等的元素。 | [immutableSortedCopy](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#immutableSortedCopy%28java.lang.Iterable%29) |
| [min\(E, E\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#min%28E, E%29) | 返回两个参数中最小的那个。如果相等，则返回第一个参数。 | [max\(E, E\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#max%28E, E%29) |
| [min\(E, E, E, E...\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#min%28E, E, E, E...%29) | 返回多个参数中最小的那个。如果有超过一个参数都最小，则返回第一个最小的参数。 | [max\(E, E, E, E...\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#max%28E, E, E, E...%29) |
| [min\(Iterable\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#min%28java.lang.Iterable%29) | 返回迭代器中最小的元素。如果可迭代对象中没有元素，则抛出NoSuchElementException。 | [max\(Iterable\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#max%28java.lang.Iterable%29),[min\(Iterator\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#min%28java.util.Iterator%29),[max\(Iterator\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#max%28java.util.Iterator%29) |

---

## 1.5-Throwables：简化异常和错误的传播与检查




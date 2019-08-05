# 1.1[使用和避免null](http://ifeve.com/using-and-avoiding-null/)

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




## **Optional**

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




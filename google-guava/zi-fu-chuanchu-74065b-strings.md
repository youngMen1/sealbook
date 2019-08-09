## 6-字符串处理：分割，连接，填充

## 连接器\[Joiner\]

用分隔符把字符串序列连接起来也可能会遇上不必要的麻烦。如果字符串序列中含有null，那连接操作会更难。Fluent风格的[Joiner](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Joiner.html)让连接字符串更简单。

```
Joiner joiner = Joiner.on("; ").skipNulls();
return joiner.join("Harry", null, "Ron", "Hermione");
```

上述代码返回”Harry; Ron; Hermione”。另外，useForNull\(String\)方法可以给定某个字符串来替换null，而不像skipNulls\(\)方法是直接忽略null。 Joiner也可以用来连接对象类型，在这种情况下，它会把对象的toString\(\)值连接起来。

```
Joiner.on(",").join(Arrays.asList(1, 5, 7)); // returns "1,5,7"
```

警告：joiner实例总是不可变的。用来定义joiner目标语义的配置方法总会返回一个新的joiner实例。这使得joiner实例都是线程安全的，你可以将其定义为static final常量。

## 拆分器\[Splitter\]

JDK内建的字符串拆分工具有一些古怪的特性。比如，String.split悄悄丢弃了尾部的分隔符。 问题：”,a,,b,”.split\(“,”\)返回？

1. “”, “a”, “”, “b”, “”
2. null, “a”, null, “b”, null
3. “a”, null, “b”
4. “a”, “b”
5. 以上都不对

正确答案是5：””, “a”, “”, “b”。只有尾部的空字符串被忽略了。[Splitter](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html)使用令人放心的、直白的流畅API模式对这些混乱的特性作了完全的掌控。

```
Splitter.on(',')
        .trimResults()
        .omitEmptyStrings()
        .split("foo,bar,,   qux");
```

上述代码返回Iterable&lt;String&gt;，其中包含”foo”、”bar”和”qux”。Splitter可以被设置为按照任何模式、字符、字符串或字符匹配器拆分。

### 拆分器工厂

| **方法** | **描述** | **范例** |
| :--- | :--- | :--- |
| [Splitter.on\(char\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#on%28char%29) | 按单个字符拆分 | Splitter.on\(‘;’\) |
| [Splitter.on\(CharMatcher\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#on%28com.google.common.base.CharMatcher%29) | 按字符匹配器拆分 | Splitter.on\(CharMatcher.BREAKING\_WHITESPACE\) |
| [Splitter.on\(String\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#on%28java.lang.String%29) | 按字符串拆分 | Splitter.on\(“,   “\) |
| [Splitter.on\(Pattern\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#on%28java.util.regex.Pattern%29)[Splitter.onPattern\(String\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#onPattern%28java.lang.String%29) | 按正则表达式拆分 | Splitter.onPattern\(“\r?\n”\) |
| [Splitter.fixedLength\(int\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#fixedLength%28int%29) | 按固定长度拆分；最后一段可能比给定长度短，但不会为空。 | Splitter.fixedLength\(3\) |

### 拆分器修饰符

| **方法** | **描述** |
| :--- | :--- |
| [omitEmptyStrings\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#omitEmptyStrings%28%29) | 从结果中自动忽略空字符串 |
| [trimResults\(\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#trimResults%28%29) | 移除结果字符串的前导空白和尾部空白 |
| [trimResults\(CharMatcher\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#trimResults%28com.google.common.base.CharMatcher%29) | 给定匹配器，移除结果字符串的前导匹配字符和尾部匹配字符 |
| [limit\(int\)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#limit%28int%29) | 限制拆分出的字符串数量 |

如果你想要拆分器返回List，只要使用Lists.newArrayList\(splitter.split\(string\)\)或类似方法。警告：splitter实例总是不可变的。用来定义splitter目标语义的配置方法总会返回一个新的splitter实例。这使得splitter实例都是线程安全的，你可以将其定义为static final常量。


## 字段上移

两个subclasses 拥有相同的值域。

**将此一值域移至superclass。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig01.gif)

**动机（Motivation）**

如果各个subclass 是分别开发的，或者是在重构过程中组合起来的，你常会发现它们拥有重复特性，特别是值域更容易重复。这样的值域有时拥有近似的名字，但也并非绝对如此。判断若干值域是否重复，惟一的办法就是观察函数如何使用它们。如果它们被使用的方式很相似，你就可以将它们归纳到superclass 去。

本项重构从两方面减少重复：首先它去除了「重复的数据声明」；其次它使你可以将使用该值域的行为从subclass 移至superclass，从而去除「重复的行为」。

**作法（Mechanics）**

* 针对待提升之值域，检查它们的所有被使用点，确认它们以同样的方式被使用。
* 如果这些值域的名称不同，先将它们改名，使每一个名称都和你想为superclass 值域取的名称相同。
* 编译，测试。
* 在superclass 中新建一个值域。
  * 如果这些值域是private ，你必须将superclass 的值域声明为protected，这样subclasses 才能引用它。
* 移除subclass 中的值域。
* 编译，测试。
* 考虑对superclass 的新建值域使用
  [封装值域](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_7)。

---

## 函数上移

有些函数，在各个subclass 中产生完全相同的结果。

**将该函数移至superclass。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig02.gif)

**动机（Motivation）**

避免「行为重复」是很重要的。尽管「重复的两个函数」也可以各自工作得很好， 但「重复」自身会成为错误的滋生地，此外别无价值。无论何时，只要系统之内出现重复，你就会面临「修改其中一个却未能修改另一个」的风险。通常，找出重复也有一定困难。

如果某个函数在各subclass 中的函数体都相同（它们很可能是通过「拷贝-粘贴」得到的），这就是最显而易见的[函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)适用场合。当然，情况并不总是如此明显。你也可以只管放心地重构，再看看测试程序会不会发牢骚，但这就需要对你的测试有充分的信心。我发现，观察这些可疑（可能重复的〕函数之间的差异往往大有收获：它们经常会向我展示那些我忘记测试的行为。

[函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)常常紧随其他重构而被使用。也许你能找出若干个「身处不 同subclasses 内的函数」而它们又可以「通过某种形式的参数调整」而后成为相同函数。这时候，最简单的办法就是首先分别调整这些函数的参数，然后再将它们概括（generalize）到superclass中。当然，如果你自信足够，也可以一次同时完成这两个步骤。

有一种特殊情况也需要使用[函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)： subclass 的函数覆写（overrides） 了superclass 的函数，但却仍然做相同的工作。

[函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)过程中最麻烦的一点就是：被提升的函数可能会引用「只出现于subclass 而不出现于superclass」的特性。如果被引用的是个函数，你可以将该函数也一同提升到superclass，或者在superclass 中建立一个抽象函数。在此过程中，你可能需要修改某个函数的签名式（signature），或建立一个委托函数（delegating method）。

如果两个函数相似但不相同，你或许可以先以[塑造模板函数](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_5)构造出相同的函数，然后再提升它们。

**作法（Mechanics）**

* 检查「待提升函数」，确定它们是完全一致的（identical）。
  * 如果这些函数看上去做了相同的事，但并不完全一致，可使用
    [替换你的算法](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_10)
    让它们变得完全一致。
* 如果「待提升函数」的签名式（signature）不同，将那些签名式都修改为你想要在superclass 中使用的签名式。
* 在superclass 中新建一个函数，将某一个「待提升函数」的代码拷贝到其中，做适当调整，然后编译。
  * 如果你使用的是一种强型（strongly typed）语言，而「待提升函数」 又调用了一个「只出现于subclass 未出现于superclass」的函数，你可以在superclass 中为被调用函数声明一个抽象函数。
  * 如果「待提升函数」使用了 subclass 的一个值域，你可以使用
    [值域上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_7)
    将该值域也提升到superclass；或者也可以先使用
    [封装值域](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_7)
    ，然后在superclass 中把取值函数（getter）声明为抽象函数。
* 移除一个「待提升的subclass 函数」。
* 编译，测试。
* 逐一移除「待提升的如函数」，直到只剩下superclass 中的函数为止。每次移除之后都需要测试。
* 观察该函数的调用者，看看是否可以将它所索求的对象型别改为superclass。

---

## 构造函数本体上移

你在各个subclass 中拥有一些构造函数，它们的本体（代码）几乎完全一致。

**在superclass 中新建一个构造函数，并在subclass 构造函数中调用它。**

```
class Manager extends Employee...
    public Manager (String name, String id, int grade) {
        _name = name;
        _id = id;
        _grade = grade;
    }
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
public Manager (String name, String id, int grade) {
    super (name, id);
    _grade = grade;
}
```

## 函数下移

## 字段下移

## 提炼子类

## 提炼超类

## 提炼接口

## 折叠继承体系

## 塑造模板函数 {#_5}

## 以委托取代继承 {#_12}

## 以继承取代委托 {#_11}




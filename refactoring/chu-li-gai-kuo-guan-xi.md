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

**动机（Motivation）**

构造函数（constructors ）是很奇妙的东西。它们不是普通函数，使用它们比使用普通函数受到更多的限制。

如果你看见各个subclass 中的函数有共同行为，你的第一个念头应该是将共同行为提炼到一个独立函数中，然后将这个函数提升到superclass 。对构造函数而言，它们彼此的共同行为往往就是「对象的建构」。这时候你需要在superclass 中提供一个构造函数，然后让subclass 都来调用它。很多时候，「调用superclass 构造函数」就是subclass 构造函数的惟一动作。这里不能运用[函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)，因为你无法在subclass 中继承superclass 构造函数（你可曾痛恨过这个规定？）。

如果重构过程过于复杂，你可以考虑转而使用[以工厂函数取代构造函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_10)。

**作法（Mechanics）**

* 在superclass 中定义一个构造函数。
* 将subclass 构造函数中的共同代码搬移到superclass 构造函数中。
  * 被搬移的可能是subclass 构造函数的全部内容。
  * 首先设法将共同代码搬移到subclass 构造函数起始处，然后再拷贝到superclass构造函数中。
* 将subclass 构造函数中的共同代码删掉，改而调用新建的superclass 构造函数。
  * 如果subclass 构造函数中的所有代码都是共同码，那么对superclass 构造函数的调用将是subclass 构造函数的惟一动作。
* 编译，测试。
  * 如果日后subclass 构造函数再出现共同代码，你可以首先使用
    [提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)
    将那一部分提炼到一个独立函数，然后使用
    [函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)
    将该函数上移到superclass。

---

## 函数下移

superclass 中的某个函数只与部分（而非全部）subclasses 有关。

**将这个函数移到相关的那些subclasses 去。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig05.gif)

**动机（Motivation）**

[函数下移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_10)恰恰相反于[函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)。当我有必要把某些行为从superclass 移至特定的subclass 时，我就使用[函数下移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_10)，它通常也只在这种时候有用。使用[提炼子类](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_3)之后你可能会需要它。

**作法（Mechanics）**

* 在所有subclass 中声明该函数，将superclass 中的函数本体拷贝到每一个subclass 函数中。
  * 你可能需要将superclass 的某些值域声明为protected，让subclass 函数也能够访问它们。如果日后你也想把这些值域下移到subclasses ， 通常就可以那么做；否则应该使用superclass 提供的访问函数（accessors）。如果访问函数并非public ，你得将它声明为protected 。
* 删除superclass 中的函数。
  * 你可能必须修改调用端的某些变量声明或参数声明，以便能够使用subclass 。
  * 如果有必要通过一个superclass 对象访问该函数，或如果你不想把该函数从任何subclass 中移除，或如果superclass 是抽象类，那么你就可以在superclass 中把该函数声明为抽象函数。
* 编译，测试。
* 将该函数从所有不需要它的那些subclasses 中删掉。
* 编译，测试。

---

## 字段下移

superclass 中的某个值域只被部分（而非全部）subclasses 用到。

**将这个值域移到需要它的那些subclasses 去。**![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig06.gif)

**动机（Motivation）**

[值域下移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_9)恰恰相反[值域上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_7)。如果只有某些（而非全部）subclasses 需要superclass 内的一个值域，你可以使用本项重构。

**作法（Mechanics）**

* 在所有subclass 中声明该值域。
* 将该值域从superclass 中移餘。
* 编译，测试。
* 将该值域从所有不需要它的那些subclasses 中删掉。
* 编译，测试。

---

## 提炼子类

class 中的某些特性（features）只被某些（而非全部）实体（instances）用到。

**新建一个subclass ，将上面所说的那一部分特性移到subclass 中。**![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig07.gif)

**动机（Motivation）**

使用[提炼子类](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_3)的主要动机是：你发现class 中的某些行为只被一部分实体用到，其他实体不需要它们。有时候这种行为上的差异是通过type code 区分 的，此时你可以使用[以子类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_14)或[以State/Strategy取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#statestrategy)。但是，并非一定要出现了type code 才表示需要考虑使用subclass 。

[提炼类](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_1)是[提炼子类](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_3)之外的另一种选择，两者之间的抉择其实就是委托（delegation）和继承（inheritance）之间的抉择。[提炼子类](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_3)通常更容易进行，但它也有限制：一旦对象创建完成，你无法再改变「与型别相关的行为」（class-based behavior ）。但如果使用[提炼类](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_1)，你只需插入另一个不同组件（ plugging in different components）就可以改变对象的行为。此外，subclasses 只能用以表现一组变化（one set of variations）。如果你希望class 以数种不同的方式变化，就必须使用委托（delegation）。

**作法（Mechanics）**

* 为source class 定义一个新的subclass 。
* 为这个新的subclass 提供构造函数。
  * 简单的作法是：让subclass 构造函数接受与superclass 构造函数相同的参数，并通过super 调用superclass 构造函数。
  * 如果你希望对用户隐藏subclass 的存在，可使用
    [以工厂函数取代构造函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_10)
    。
* 找出调用superclass 构造函数的所有地点。如果它们需要的是新建的subclass ， 令它们改而调用新构造函数。
  * 如果subclass 构造函数需要的参数和superclass 构造函数的参数不同，可以使用
    [重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)
    修改其参数列。如果subclass 构造函数不需要superclass 构造函数的某些参数，可以使用
    [重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)
    将它们去除。
  * 如果不再需要直接实体化（具现化，instantiated）superclass ，就将它声明为抽象类。
* 逐一使用
  [函数下移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_10)
  和
  [值域下移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_9)
  将source class 的特性移到subclass 去。
  * 和
    [提炼类](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_1)
    不同的是，先处理函数再处理数据，通常会简单一些。
  * 当一个public 函数被下移到subclass 后，你可能需要重新定义该函数的调用端的局部变量或参数型别，让它们改调用subclass 中的新函数。如果忘记进行这一步骤，编译器会提醒你。
* 找到所有这样的值域：它们所传达的信息如今可由继承体系自身传达（这一类值域通常是boolean 变量或type code ）。以
  [封装值域](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_7)
  避免直接使用这些值域，然后将它们的取值函数（getter）替换为多态常量函数（polymorphic constant methods）。所有使用这些值域的地方都应该以
  [以多态取代条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_6)
  重构。
  * 任何函数如果位于source class 之外，而又使用了上述值域的访问函数（accessors），考虑以
    [搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)
    将它移到source class 中， 然后再使用
    [以多态取代条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_6)
    。
* 每次下移之后，编译并测试。

---

## 提炼超类

两个classes 有相似特性（similar features）。

**为这两个classes 建立一个superclass ，将相同特性移至superclass 。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig08.gif)

**动机（Motivation）**

重复代码是系统中最主要的一种糟糕东西。如果你在不同的地方进行相同一件事 情，一旦需要修改那些动作时，你就得负担比你原本应该负担的更多事情。

重复代码的某种形式就是：两个classes 以相同的方式做类似的事情，或者以不同的方式做类似的事情。对象提供了一种简化这种情况的机制，那就是继承机制。但是，在建立这些具有共通性的classes 之前，你往往无法发现这样的共通性，因此你经常会在「具有共通性」的classes 存在之后，再幵始建立其间的继承结构。

另一种选择就是[提炼类](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_1)。这两种方案之间的选择其实就是继承（Inheritance ）和委托（delegation）之间的选择。如果两个classes 可以共享行为， 也可以共享接口，那么继承是比较简单的作法。如果你选错了，也总有[以委托取代继承](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_12)这瓶后悔药可吃。

**作法（Mechanics）**

* 为原本的classes 新建一个空白的abstract superclass。
* 运用
  [值域上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_7)
  ,
  [函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)
  , 和
  [构造函数本体上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_6)
  逐一将subclass 的共同充素上移到superclass 。
  * 先搬移值域，通常比较简单。
  * 如果相应的subclass 函数有不同的签名式（signature），但用途相同，可以先使用
    [重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)
    将它们的签名式改为相同，然后 再使用
    [函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)
    。
  * 如果相应的subclass 函数有相同的签名式，但函数本体不同，可以在superclass 中把它们的共同签名式声明为抽象函数。
  * 如果相应的subclass 函数有不同的函数本体，但用途相同，可试着使用
    [替换你的算法](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_10)
    把其中一个函数的函数本体拷贝到另一个函数中。如果运转正常，你就可以使用
    [函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)
    。
* 每次上移后，编译并测试。
* 检查留在subclass 中的函数，看它们是否还有共通成分。如果有，可以使用
  [提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)
  将共通部分再提炼出来，然后使用
  [函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)
  将提炼出的函数上移到superclass 。如果各个subclass 中某个函数的整体流程很相似，你也许可以使用
  [塑造模板函数](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_5)
  。
* 将所有共通元素都上移到superclass 之后，检查subclass 的所有用户。如果它们只使用共同接口，你就可以把它们所索求的对象型别改为superclass 。

---

## 提炼接口

若干客户使用class 接口中的同一子集；或者，两个classes 的接口有部分相同。

**将相同的子集提炼到一个独立接口中。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig09.gif)

**动机（Motivation）**

classes 之间彼此互用的方式有若干种。「使用一个class 」通常意味覆盖该class 的所有责任区（ whole area of responsibilities ）。另一种情况是，某一组客户只使用class 责任区中的一个特定子集。再一种情况则是，class 需要与「所有可协助处理某些特定请求」的classes 合作。

对于后两种情况，将「被使用之部分责任」分离出来通常很有意义，因为这样可以使系统的用法更清晰，同时也更容易看清系统的责任划分。如果新的需要支持上述子集，也比较能够看清子集内有些什么东西。

在许多面向对象语言中，这种「责任划分」能力是通过多重继承（multiple inheritance）支持的。你可以针对一段行为（each segment of behavior ）建立一个class ，再将它们组合于一份实现品（implementation）中。Java 只提供单一继承（single inher），但你可以运用interfaces \(接口〉来昭示并实现上述需求。interfaces 对于Java 程序的设计方式有着巨大的影响，就连Smalltalk 程序员都认为interfaces （接口） 是一大进步！

[提炼超类](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_4)和[提炼接口](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_2)之间有些相似之处。[提炼接口](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_2)只能提炼共通接口，不能提炼共通代码。使用[提炼接口](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_2)可能造成难闻的「重复」臭味，幸而你可以运用[提炼类](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_1)先把共通行为放进一个组件（component）中，然后将工作委托（delegating）该组件，从而解决这个问题。如果有不少共通行为，[提炼超类](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_4)会比较简单，但是每个class 只能有一个superclass（译注：每个class 却能有多个interfaces ）。

如果某个class 在不同环境下扮演截然不同的角色，使用interface （接口）就是个好主意。你可以针对每个角色以[提炼接口](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_2)提炼出相应接口。另一种可以用上[提炼接口](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_2)的情况是：你想要描述一个class 的外驶接口（outbound interface ），亦即这个class 对其server 所进行的操作〉。如果你打算将来加入其他种类的server ，只需要求它们实现这个接口即可。

**作法（Mechanics）**

* 新建一个空接口（empty interface ）。
* 在接口中声明「待提炼类」的共通操作。
* 让相关的胡实现上述接口。
* 调整客户端的型别声明，使得以运用该接口。

---

## 折叠继承体系

superclass 和subclass 之间无太大区别。

**将它们合为一体。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig10.gif)

**动机（Motivation）**

如果你曾经编写过继承体系，你就会知道，继承体系很容易变得过分复杂。所谓重构继承体系，往往是将函数和值域在体系中上下移动。完成这些动作后，你 很可能发现某个subclass 并未带来该有的价值，因此需要把classes 并合（折叠）起来。

**作法（Mechanics）**

* 选择你想移除的class ：是superclass 还是subclass ？
* 使用
  [值域上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_7)
  和
  [函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)
  ，或者
  [函数下移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_10)
  和
  [值域下移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_9)
  ，把想要移除的class 内的所有行为和数据 （值域）搬移到另一个class 。
* 每次移动后，编译并测试。
* 调整「即将被移除的那个class 」的所有引用点，令它们改而引用合并（折叠）后留下的class 。这个动作将会影响变量的声明、参数的型别以及构造函数。
* 移除我们的目标；此时的它应该已经成为一个空类（empty class）。
* 编译，测试。

---

## 塑造模板函数 {#_5}

你有一些subclasses ，其中相应的某些函数以相同顺序执行类似的措施，但各措施实际上有所不同。

**将各个措施分别放进独立函数中，并保持它们都有相同的签名式（signature），于是原函数也就变得相同了。然后将原函数上移至superclass 。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig11.gif)

**动机（Motivation）**

继承是「避免重复行为」的一个强大工具。无论何时，只要你看见两个subclasses 之中有类似的函数，就可以把它们提升到superclass 。但是如果这些函数并不完全相同呢？此时的你应该怎么办？我们仍有必要尽量避免重复，但又必须保持这些函 数之间的实质差异。

常见的一种情况是：两个函数以相同序列（sequence）执行大致相近的措施，但是各措施不完全相同。这种情况下我们可以将「执行各措施」的序列移至superclass ， 并倚赖多态（polymorphism ）保证各措施仍得以保持差异性。这样的函数被称为Template Method （模板函数）\[Gang of Four\]。

**作法（Mechanics）**

* 在各个subclass 中分解目标函数，使分解后的各个函数要不完全相同，要不完全不同。
* 运用
  [函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)
  将各subclass 内寒全相同的函数上移至superclass 。
* 对于那些（剩余的、存在于各subclasses 内的）完全不同的函数，实施
  [重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)
  ，使所有这些函数的签名式（signature）完全相同。
  * 这将使得原函数变为完全相同，因为它们都执行同样一组函数调用； 但各subclass 会以不同方式响应这些调用。
* 修改上述所有签名式后，编译并测试。
* 运用
  [函数上移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_8)
  将所有原函数上移至superclass 。在superclass 中将那些「有所不同、代表各种不同措施」的函数定义为抽象函数。
* 编译，测试。
* 移除其他subclass 中的原函数，每删除一个，编译并测试。

---

## 以委托取代继承 {#_12}

某个subclass 只使用superclass 接口中的一部分，或是根本不需要继承而来的数据。

**在subclass 中新建一个值域用以保存superclass ；调整subclass 函数，令它改而委托superclass ；然后去掉两者之间的继承关系。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig14.gif)

**动机（Motivation）**

继承（Inheritance ）是一件很棒的事，但有时候它并不是你要的。常常你会遇到这样的情况：一开始你继承了一个class ，随后发现superclass 的许多操作并不真正 适用于subclass 。这种情况下你所拥有的接口并未真正反映出class 的功能。或者，你可能发现你从superclass 中继承了 一大堆subclass 并不需要的数据，抑或者你可能发现superclass 中的某些protected 函数对subclass 并没有什么意义。

你可以选择容忍，并接受传统说法：subclass 可以只使用superclass 功能的一部分。但这样做的结果是：代码传达的信息与你的意图南辑北辙——这是一种裩淆，你应该将它去除。

如果以委托（delegation）取代继承（Inheritance ），你可以更清楚地表明：你只需要受托类（delegated class）的一部分功能。接口中的哪一部分应该被使用，哪一部分应该被忽略，完全由你主导控制。这样做的成本则是需要额外写出请托函数（delegating methods），但这些函数都非常简单，极少可能出错。

**作法（Mechanics）**

* 在subclass 中新建一个值域，使其引用（指向、指涉、refers）superclass 的一个实体，并将它初始化为this。
* 修改subclass 内的每一个（可能）函数，让它们不再使用superclass ，转而使 用上述那个「受托值域」（delegated field）。每次修改后，编译并测试。
  * 你不能如此这般地修改subclass 中「通过super 调用superclass 函数」的函数，否则它们会陷入无限递归（infinite recurse）。这一类函数只有在继承关系被打破后才能修改。
* 去除两个classes 之间的继承关系，将上述「受托值域」（delegated field）的赋值动作修改为「赋予一个新对象」。
* 针对客户端所用的每一个superclass 函数，为它添加一个简单的请托函数（delegating method）。
* 编译，测试。

---

## 以继承取代委托 {#_11}

你在两个classes 之间使用委托关系（delegation），并经常为整个接口编写许多极简单的请托函数（delegating methods）。

**让「请托（delegating）class」继承「受托 class （delegate）」。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/11fig15.gif)

**动机（Motivation）**

本重构与[以继承取代委托](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_11)恰恰相反。如果你发现自己需要使用「受托 class」中的所有函数，并且费了很大力气编写所有极简的请托函数（delegating methods），本重构可以帮助你轻松回头使用「继承」。

两条告诫需牢记于心。首先，如果你并没有使用「受托class 」的所有函数（而非只是部分函数），那么就不应该使用[以继承取代委托](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_11)，因为subclass 应该总是遵循（奉行）superclass 的接口。如果过多的请托函数让你烦心，你有别的选择：你可以通过[移除中间人](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_6)让客户端自己调用受托函数，也可以使用[提炼超类](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_4)将两个classes 接口相同的部分提炼到superclass 中， 然后让两个classes 都继承这个新的superclass ；你还可以以类似手法使用[提炼接口](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_2)。

另一种需要当心的情况是：受托对象被不止一个其他对象共享，而且受托对象是可变的（mutable）。在这种情况下，你就不能将「委托关系」替换为「继承关系」，因为这样就无法再共享数据了。数据共享是必须由「委托关系」承担的一种责任，你无法把它转给「继承关系」。如果受托对象是不可变的（immutable），数据共享就不 成问题，因为你大可放心地拷贝对象，谁都不会知道。

**作法（Mechanics）**

* 让「请托端」成为「受托端」的一个subclass 。
* 编译。
  * 此时，某些函数可能会发生冲突：它们可能有相同的名称，但在返回型别（return type）、异常指定（exceptions）或可视性（visibility）方面有所差异。你可以使用
    [重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)
    解决此类问题。
* 将「受托值域」（delegated field）设为「该值域所处之对象自身」。
* 去掉简单的请托函数（delegating methods）。
* 编译并测试。
* 将所有其他「涉及委托关系」的动作，改为「调用对象自身（继承而来的函 数）」。
* 移除「受托值域」（delegated field）。




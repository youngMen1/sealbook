## 搬移涵数

（译注：本节大量保留class,method,source,target等字眼）

**你的程序中，有个函数与其所驻class之外的另一个class进行更多交流：调用后者，或被后者调用。**

在该函数最常引用（指涉）的class中建立一个有着类似行为的新函数。将旧函数变成一个单纯的委托函数（delegating method），或是将旧函数完全移除。

![](/assets/微信截图_20190726164400.png)

**动机（Motivation）**

「函数搬移」是重构理论的支柱。如果一个class有太多行为，或如果一个class与另一个class有太多合作而形成高度耦合（highly coupled\)，我就会搬移函数。通过这种手段，我可以使系统中的classes更简单，这些classes最终也将更干净利落地实现系统交付的任务。

常常我会浏览class的所有函数，从中寻找这样的函数：使用另一个对象的次数比使用自己所驻对象的次数还多。一旦我移动了一些值域，就该做这样的检查。一旦发现「有可能被我搬移」的函数，我就会观察调用它的那一端、它调用的那一端，以及继承体系中它的任何一个重定义函数。然后，我会根据「这个函数与哪个对象的交流比较多」，决定其移动路径。

这往往不是一个容易做出的决定。如果不能肯定是否应该移动一个函数，我就会继续观察其他函数。移动其他函数往往会让这项决定变得容易一些。有时候，即使你移动了其他函数，还是很难对眼下这个函数做出决定。其实这也没什么大不了的。 如果真的很难做出决定，那么或许「移动这个函数与否」并不那么重要。所以，我会凭本能去做，反正以后总是可以修改的。

**做法（Mechanics）**

* 检查source class定义之source method所使用的一切特性（features），考虑它们是否也该被搬移。（译注：此处所谓特性泛指class定义的所有东西，包括值域和函数。）
  * 如果某个特性只被你打算搬移的那个函数用到，你应该将它一并搬移。如果另有其他函数使用了这个特性，你可以考虑将使用该特性的所有函数全都一并搬移。有时候搬移一组函数比逐一搬移简单些。
* 检查source class的subclass和superclass，看看是否有该函数的其他声明。
  * 如果出现其他声明，你或许无法进行搬移，除非target class也同样表现出多态性（polylmorphism〕。
* 在target class中声明这个函数。
  * 你可以为此函数选择一个新名称——对target class更有意义的名称。
* 将source method的代码拷贝到target method中。调整后者，使其能在新家中正常运行。
  * 如果target method使用了source特性，你得决定如何从target method引用source object。如果target class中没有相应的引用机制，就把source object reference当作参数，传给新建立的target class。
  * 如果source method包含异常处理式（exception handler），你得判断逻辑上应该由哪个来处理这一异常。如果应该由source class负责，就把异常处理式留在原地。
* 编译target class。
* 决定如何从source正确引用target object。
  * 可能会有一个现成的值域或函数帮助你取得target class。如果没有，就看能否轻松建立一个这样的函数。如果还是不行，你得在source class中新建一个新值域来保存target object。这可能是一个永久性修改，但你也可以让它保持暂时的地位，因为后继的其他重构项目可能会把这个新建值域去掉。
* 修改source method，使之成为一个delegating method（纯委托函数〕。
* 编译，测试。
* 决定「删除source method」或将它当作一个delegating method保留下来。
  * 如果你经常要在source object中引用target method，那么将source method作为delegating method保留下来会比较简单。
  * 如果你想移除source method，请将source class中对source method的所有引用动作，替换为「对target method的引用动作」。
* 编译，测试。

---

## 搬移字段

译注：本节大量保留class,method,source,target等字眼）

你的程序中，某个field（值域〕被其所驻class之外的另一个class更多地用到。

**在target class 建立一个new field，修改source field的所有用户，令它们改用此new field。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/07fig02.gif)

**动机（Motivation）**

在classes之间移动状态（states）和行为，是重构过程中必不可少的措施。随着系统发展，你会发现自己需要新的class，并需要将原本的工作责任拖到新的class中。这个星期中合理而正确的设计决策，到了下个星期可能不再正确。这没问题；如果你从来没遇到这种情况，那才有问题。

如果我发现，对于一个field（值域），在其所驻class之外的另一个class中有更多函数使用了它，我就会考虑搬移这个field。上述所谓「使用」可能是通过设值/取值（setting/getting）函数间接进行。我也可能移动该field的用户（某函数），这取决于是否需要保持接口不受变化。如果这些函数看上去很适合待在原地，我就选择搬移field。

使用[提炼类](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_1)时，我也可能需要搬移field。此时我会先搬移field，然后再搬移函数。

**做法（Mechanics）**

* 如果field的属性是public，首先使用
  [封装值域](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_7)
  将它封装起来。
* 如果你有可能移动那些频繁访问该field的函数，或如果有许多函数访问某个field，先使用
  [封装值域](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_7)
  也许会有帮助。
* 编译，测试。
* 在target class中建立与source field相同的field，并同时建立相应的设值/取值 （setting/getting）函数。
* 编译target class。
* 决定如何在source object中引用target object。
* 一个现成的field或method可以助你得到target object。如果没有，就看能否轻易建立这样一个函数。如果还不行，就得在source class中新建一个field来存放target object。这可能是个永久性修改，但你也可以暂不公开它，因为后续重构可能会把这个新建field除掉。
* 删除source field。
* 将所有「对source field的引用」替换为「对target适当函数的调用」。
* 如果是「读取」该变量，就把「对source field的引用」替换为「对target取值函数（getter）的调用」；如果是「赋值」该变量，就把对source field的引用」替换成「对设值函数（setter）的调用」。
* 如果source field不是private，就必须在source class的所有subclasses中查找source field的引用点，并进行相应替换。
* 编译，测试。

**范例（Examples）**

下面是Account class的部分代码：

```
class Account...
    private AccountType _type;
    private double _interestRate;

    double interestForAmount_days (double amount, int days) {
        return _interestRate * amount * days / 365;
    }
```

我想把表示利率的\_interestRate搬移到AccountType class去。目前已有数个函数引用了它，interestForAmount\_days\(\) 就是其一。下一步我要在AccountType中建立\_interestRate field以及相应的访问函数：

```
class AccountType...
    private double _interestRate;

    void setInterestRate (double arg) {
        _interestRate = arg;
    }

    double getInterestRate () {
        return _interestRate;
    }
```

这时候我可以编译新的AccountType class。

现在，我需要让Account class中访问此\_interestRate field的函数转而使用AccountType对象，然后删除Account class中的\_interestRate field。我必须删除source field，才能保证其访问函数的确改变了操作对象，因为编译器会帮我指出未正确获得修改的函数。

```
private double _interestRate;
double interestForAmount_days (double amount, int days) {
    return _type.getInterestRate() * amount * days / 365;
}
```

---

## 提炼类

某个class做了应该由两个classes做的事。

**建立一个新class，将相关的值域和函数从旧class搬移到新class。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/07fig03.gif)

**动机（Motivation）**

你也许听过类似这样的教诲：一个class应该是一个清楚的抽象（abstract），处理一些明确的责任。但是在实际工作中，class会不断成长扩展。你会在这儿加入一些功能，在那儿加入一些数据。给某个class添加一项新责任时，你会觉得不值得为这项责任分离出一个单独的class。于是，随着责任不断増加，这个class会变得过份复杂。很快，你的class就会变成一团乱麻。

这样的class往往含有大量函数和数据。这样的class往往太大而不易理解。此时你需要考虑哪些部分可以分离出去，并将它们分离到一个单独的class中。如果某些数据和某些函数总是一起出现，如果某些数据经常同时变化甚至彼此相依，这就表示你应该将它们分离出去。一个有用的测试就是问你自己，如果你搬移了某些值域和函数，会发生什么事？其他值域和函数是否因此变得无意义？

另一个往往在开发后期出现的信号是class的「subtyped方式」。如果你发现subtyping只影响class的部分特性，或如果你发现某些特性「需要以此方式subtyped」，某些特性「需要以彼方式subtyped」，这就意味你需要分解原来的class。

**做法（Mechanics）**

* 决定如何分解cass所负责任。
* 建立一个新class，用以表现从旧class中分离出来的责任。
* 如果旧class剩下的责任与旧class名称不符，为旧class易名。
* 建立「从旧class访问新class」的连接关系（link）。
* 也许你有可能需要一个双向连接。但是在真正需要它之前，不要建立 「从新class通往旧class」的连接。
* 对于你想搬移的每一个值域，运用[搬移值域](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_2)搬移之。
* 每次搬移后，编译、测试。
* 使用[搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)将必要函数搬移到新class。先搬移较低层函数（也就是「被其他函数调用」多于「调用其他函数」者），再搬移较高层函数。
* 每次搬移之后，编译、测试。
* 检查，精简每个class的接口。
* 如果你建立起双向连接，检查是否可以将它改为单向连接。
* 决定是否让新class曝光。如果你的确需要曝光它，决定让它成为reference object \(引用型对象〕或immutable value object（不可变之「实值型对象」）。

**范例（Examples）**

让我们从一个简单的Person class开始：

```
class Person...
    public String getName() {
        return _name;
    }
    public String getTelephoneNumber() {
        return ("(" + _officeAreaCode + ") " + _officeNumber);
    }
    String getOfficeAreaCode() {
        return _officeAreaCode;
    }
    void setOfficeAreaCode(String arg) {
        _officeAreaCode = arg;
    }
    String getOfficeNumber() {
        return _officeNumber;
    }
    void setOfficeNumber(String arg) {
        _officeNumber = arg;
    }

    private String _name;
    private String _officeAreaCode;
    private String _officeNumber;
```

在这个例子中，我可以将「与电话号码相关」的行为分离到一个独立class中。首 先我耍定义一个TelephoneNumber class来表示「电话号码」这个概念：

```
class TelephoneNumber {
}
```

易如反掌！然后，我要建立从Person到TelephoneNumber的连接：

```
class Person
    private TelephoneNumber _officeTelephone = new TelephoneNumber();
```

现在，我运用[搬移值域](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_2)移动一个值域：

```
class TelephoneNumber {
    String getAreaCode() {
        return _areaCode;
    }
    void setAreaCode(String arg) {
        _areaCode = arg;
    }
    private String _areaCode;
}

class Person...
    public String getTelephoneNumber() {
        return ("(" + getOfficeAreaCode() + ") " + _officeNumber);
    }
    String getOfficeAreaCode() {
        return _officeTelephone.getAreaCode();
    }
    void setOfficeAreaCode(String arg) {
        _officeTelephone.setAreaCode(arg);
    }
```

然后我可以移动其他值域，并运用[搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)将相关函数移动到TelephoneNumber class中：

```
class Person...
    public String getName() {
        return _name;
    }
    public String getTelephoneNumber(){
        return _officeTelephone.getTelephoneNumber();
    }
    TelephoneNumber getOfficeTelephone() {
        return _officeTelephone;
    }

    private String _name;
    private TelephoneNumber _officeTelephone = new TelephoneNumber();

class TelephoneNumber...
    public String getTelephoneNumber() {
        return ("(" + _areaCode + ") " + _number);
    }
    String getAreaCode() {
        return _areaCode;
    }
    void setAreaCode(String arg) {
        _areaCode = arg;
    }
    String getNumber() {
        return _number;
    }
    void setNumber(String arg) {
        _number = arg;
    }
    private String _number;
    private String _areaCode;
```

下一步要做的决定是：要不要对客户揭示这个新口class？我可以将Person中「与电 话号码相关」的函数委托（delegating）至TelephoneNumber，从而完全隐藏这个新class；也可以直接将它对用户曝光。我还可以将它暴露给部分用户（位于同一个package中的用户），而不暴露给其他用户。

如果我选择暴露新class，我就需要考虑别名（aliasing）带来的危险。如果我暴露了TelephoneNumber ，而有个用户修改了对象中的\_areaCode值域值，我又怎么能知道呢？而且，做出修改的可能不是直接用户，而是用户的用户的用户。

面对这个问题，我有下列数种选择：

* 允许任何对象修改TelephoneNumber 对象的任何部分。这就使得TelephoneNumber 对象成为引用对象（reference object），于是我应该考虑使用
  [将实值对象改为引用对象](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_4)
  。这种情况下，Person应该是TelephoneNumber的访问点。
* 不许任何人「不通过Person对象就修改TelephoneNumber 对象」。为了达到目的，我可以将TelephoneNumber「设为不可修改的（immutable），或为它提供一个不可修改的接口（immutable interface）。
* 另一个办法是：先复制一个TelephoneNumber 对象，然后将复制得到的新对象传递给用户。但这可能会造成一定程度的迷惑，因为人们会认为他们可以修改TelephoneNumber对象值。此外，如果同一个TelephoneNumber 对象 被传递给多个用户，也可能在用户之间造成别名（aliasing）问题。

[提炼类](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_1)是改善并发（concurrent）程序的一种常用技术，因为它使你可以为提炼后的两个classes分别加锁（locks）。如果你不需要同时锁定两个对象， 你就不必这样做。这方面的更多信息请看Lea\[Lea\]， 3.3节。

这里也存在危险性。如果需要确保两个对象被同时锁定，你就面临事务（transaction）问题，需要使用其他类型的共享锁〔shared locks〕。正如Lea\[Lea\] 8.1节所讨论， 这是一个复杂领域，比起一般情况需要更繁重的机制。事务（transaction）很有实用性，但是编写事务管理程序（transaction manager）则超出了大多数程序员的职责范围。

---

## 将类内联化

的某个class没有做太多事情（没有承担足够责任）。

**将class的所有特性搬移到另一个class中，然后移除原class。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/07fig04.gif)

**动机（Motivation）**

[将类内联化](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_4)正好与[提炼类](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_1)相反。如果一个class不再承担足够 责任、不再有单独存在的理由〔这通常是因为此前的重构动作移走了这个class的 责任），我就会挑选这一「萎缩class」的最频繁用户（也是个class），以[将类内联化](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_4)手法将「妻缩class」塞进去。

**做法（Mechanics）**

* 在absorbing class（合并端的那个class）身上声明source class的public协议， 并将其中所有函数委托（delegate）至source class。
* 如果「以一个独立接口表示source class函数」更合适的话，就应该在inlining之前先使用
  [提炼接口](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_2)
  。
* 修改所有source class引用点，改而引用absorbing class。
* 将source class声明为private，以斩断package之外的所有引用可能。 同时并修改source class的名称，这便可使编译器帮助你捕捉到所有对于source class的"dangling references "（虚悬引用点）。
* 编译，测试。
* 运用
  [搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)
  和
  [搬移值域](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_2)
  ，将source class的特性全部搬移至absorbing class。
* 为source class举行一个简单的丧礼。

**范例（Examples）**

先前（上个重构项〉我从TelephoneNumber「提炼出另一个class，现在我要将它inlining塞回到Person去。一开始这两个classes是分离的：

```
class Person...
    public String getName() {
        return _name;
    }
    public String getTelephoneNumber(){
        return _officeTelephone.getTelephoneNumber();
    }
    TelephoneNumber getOfficeTelephone() {
        return _officeTelephone;
    }

    private String _name;
    private TelephoneNumber _officeTelephone = new TelephoneNumber();

class TelephoneNumber...
    public String getTelephoneNumber() {
        return ("(" + _areaCode + ") " + _number);
    }
    String getAreaCode() {
        return _areaCode;
    }
    void setAreaCode(String arg) {
        _areaCode = arg;
    }
    String getNumber() {
        return _number;
    }
    void setNumber(String arg) {
        _number = arg;
    }
    private String _number;
    private String _areaCode;
```

首先我在Person中声明TelephoneNumber「的所有「可见」（public）函数：

```
class Person...
    String getAreaCode() {
        return _officeTelephone.getAreaCode();        //译注：请注意其变化
    }
    void setAreaCode(String arg) {
        _officeTelephone.setAreaCode(arg);                //译注：请注意其变化
    }
    String getNumber() {
        return _officeTelephone.getNumber();        //译注：请注意其变化
    }
    void setNumber(String arg) {
        _officeTelephone.setNumber(arg);                //译注：请注意其变化
    }
```

现在，我要找出TelephoneNumber的所有用户，让它们转而使用Person接口。于是下列代码：

## 隐藏“委托关系”

## 移除中间人

## 引入外加函数

## 引入本地扩展




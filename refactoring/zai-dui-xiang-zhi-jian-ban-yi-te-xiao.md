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

## 提炼类

## 将类内联化

## 隐藏“委托关系”

## 移除中间人

## 引入外加函数

## 引入本地扩展




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

## 提炼类

## 将类内联化

## 隐藏“委托关系”

## 移除中间人

## 引入外加函数

## 引入本地扩展




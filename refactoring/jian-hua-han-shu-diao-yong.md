## 函数改名

函数的名称未能揭示函数的用途。**修改函数名称。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/10fig01.gif)

**动机（Motivation）**

我极力提倡的一种编程风格就是：将复杂的处理过程分解成小函数。但是，如果做得不好，这会使你费尽周折却弄不清楚这些小函数各自的用途。要避免这种麻烦，关键就在于给函数起一个好名称。函数的名称应该准确表达它的用途。给函数命名有一个好办法：首先考虑应该给这个函数写上一句怎样的注释，然后想办法将注释变成函数名称。

人生不如意，十之八九。你常常无法第一次就给函数起一个好名称。这时候你可能会想：就这样将就着吧——毕竟只是一个名称而已。当心！这是恶魔的召唤，是通 向混乱之路，千万不要被它诱惑！如果你看到一个函数名称不能很好地表达它的用 途，应该马上加以修改。记住，你的代码首先是为人写的，其次才是为计算器写的。 而人需要良好名称的函数。想想过去曾经浪费的无数时间吧。如果给每个函数都起一个良好的名称，也许你可以节约好多时间。起一个好名称并不容易，需要经验； 要想成为一个真正的编程高手，「起名称」的水平是至关重要的。当然，函数签名式（signature）中的其他部分也一样重要；如果重新安排参数顺序，能够帮助提高代码的清晰度，那就大胆地去做吧，你有[添加参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_1)和[移除参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_7)这两项武器。

**作法（Mechanics）**

* 检查函数签名式（signature）是否被superclass 或subclass 实现过。如果是，则需要针对每份实现品分别进行下列步骤。
* 声明一个新函数，将它命名为你想要的新名称。将旧函数的代码拷贝到新函数中，并进行适当调整。
* 编译。
* 修改旧函数，令它将调用转发给新函数。
  * 如果只有少数几个地方引用旧函数，你可以大胆地跳过这一步骤。
* 编译，测试。
* 找出旧函数的所有被引用点，修改它们，令它们改而引用新函数。每次修改后，编译并测试。
* 删除旧函数。
  * 如果旧函数是class public 接口的一部分，你可能无法安全地删除它。这种情况下，将它保留在原处，并将它标记为"deprecated"（不再被赞同）。
* 编译，测试。

---

## 添加参数

某个函数需要从调用端得到更多信息。**为此函数添加一个对象参数，让该对象带进函数所需信息。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/10fig02.gif)

**动机（Motivation）**

[添加参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_1)是一个很常用的重构手法，我几乎可以肯定已经用过它了。使用这项重构的动机很简单：你必须修改一个函数，而修改后的函数需要一些过去没有的信息，因此你需要给该函数添加一个参数。

实际上我比较需要说明的是：不使用本重构的时机。除了添加参数外，你常常还有其他选择。只要可能，其他选择都比本项「添加参数」要好，因为它们不会增加参数列的长度。过长的参数列是不好的味道，因为程序员很难记住那么多参数，而且长参数列往往伴随着坏味道Date Clumps。

请看看现有的参数，然后问自己：你能从这些参数得到所需的信息吗？如果回答是否定的，有可能通过某个函数提供所需信息吗？你究竟把这些信息用于何处？这个函数是否应该属于拥有该信息的那个对象所有？看看现有参数，考虑一下，加入新参数是否合适？也许你应该考虑使用[引入参数对象](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_4)。

我并非要你绝对不要添加参数。事实上我自己经常添加参数，但是在添加参数之前你有必要了解其他选择。

**作法（Mechanics）**

[添加参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_1)的作法和[重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)非常相似。

* 检查函数签名式（signature）是否被superclass 或subclass 实现过。如果是，则需要针对每份实现分别进行下列步骤。
* 声明一个新函数，名称与原函数同，只是加上新添参数。将旧函数的代码拷贝到新函数中。
  * 如果需要添加的参数不止一个，将它们一次性添加进去比较容易。
* 编译。
* 修改旧函数，令它调用新函数。
  * 如果只有少数几个地方引用旧函数，你大可放心地跳过这一步骤。
  * 此时，你可以给参数提供任意值。但一般来说，我们会给对象参数提供null ，给内置型参数提供一个明显非正常值。对于数值型参数，我建议使用0 以外的值，这样你比较容易将来认出它。
* 编译，测试。
* 找出旧函数的所有被引用点，将它们全部修改为对新函数的引用。每次修改后，编译并测试。
* 删除旧函数。
  * 如果旧函数是class public 接口的一部分，你可能无法安全地删除它。这种情况下，请将它保留在原地，并将它标示为"deprecated"（不再 被赞同）。
* 编译，测试。

---

## 移除参数

函数本体（method body）不再需要某个参数。**将该参数去除。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/10fig03.gif)

**动机（Motivation）**

程序员可能经常添加参数，却往往不愿意去掉它们。他们打的如意算盘是，无论如 何，多余的参数不会引起任何问题，而且以后还可能用上它。

这也是恶魔的诱惑，一定要把它从脑子里赶出去！参数指出函数所需信息，不同的参数值代表不同的意义。函数调用者必须为每一个参数操心该传什么东西进去。如果你不去掉多余参数，你就是让你的每一位用户多费一份心。这是很不划算的，尤其「去除参数」是非常简单的一项重构。

但是，对于多态函数（polymorphic method），情况有所不同。这种情况下，可能多态函数的另一份（或多份）实现码会使用这个参数，此时你就不能去除它。你可以添加一个独立函数，在这些情况下使用，不过你应该先检查调用者如何使用这个函数，以决定是否值得这么做。如果某些调用者已经知道他们正在处理的是 一个特定的subclass ，并且已经做了额外工作找出自己需要的参数，或已经利用对classes 体系的了解来避免取到null ，那么就值得你建立一个新函数，去除那多余参数。如果调用者不需要了解该函数所属的class ，你也可以保持调用者无知（而幸福）的状态。

**作法（Mechanics）**

[移除参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_7)的作法和[重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)、[添加参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_1)非常相似。

* 检查函数签名式（signature）是否被superclass 或如subclass 实现过。如果是，则需要针对每份实现品分别进行下列步骤。
* 声明一个新函数，名称与原函数同，只是去除不必要的参数。将旧函数的代码拷贝到新函数中。
  * 如果需要去除的参数不止一个，将它们一次性去除比较容易。
* 编译。
* 修改旧函数，令它调用新函数。
  * 如果只有少数几个地方引用旧函数，你大可放心地跳过这一步骤。
* 编译，测试。
* 找出旧函数的所有被引用点，将它们全部修改为对新函数的引用。每次修改后，编译并测试。
* 删除旧函数。
  * 如果旧函数是class public 接口的一部分，你可能无法安全地删除它。 这种情况下，将它保留在原处，并将它标记为"deprecated"（不再被赞同）。
* 编译，测试。

由于我可以轻松地添加、去除参数，所以我经常一次性地添加或去除必要的参数。

---

## 将查询函数和修改函数分离 {#_15}

某个函数既返回对象状态值，又修改对象状态（state）。**建立两个不同的函数，其中一个负责査询，另一个负责修改。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/10fig04.gif)

**动机（Motivation）**

如果某个函数只是向你提供一个值，没有任何看得到的副作用（或说连带影响）， 那么这是个很有价值的东西。你可以任意调用这个函数，也可以把调用动作搬到函 数的其他地方。简而言之，需要操心的事情少多了。

明确表现出「有副作用」与「无副作用」两种函数之间的差异，是个很好的想法。 下而是一条好规则：任何有返回值的函数，都不应该有看得到的副作用。有些程序 员甚至将此作为一条必须遵守的规则\[Meyer\]。就像对待任何东西一样，我并不绝对遵守它，不过我总是尽量遵守，而它也回报我很好的效果。

如果你遇到一个「既有返回值又有副作用」的函数，就应该试着将查询动作从修改 动作中分割出来。

你也许已经注意到了 ：我使用「看得到的副作用」这种说法。有一种常见的优化办法是：将查询所得结果高速缓存（cache）于某个值域中，这么一来后续的重复查询 就可以大大加快速度。虽然这种作法改变了对象的状态，但这一修改是察觉不到的，因为不论如何査询，你总是获得相同结果\[Meyer\]。

**做法（Mechanics）**

* 新建一个查询函数，令它返回的值与原函数相同。
  * 观察原函数，看它返回什么东西。如果返回的是一个临时变量，找出临时变量的位置。
* 修改原函数，令它调用查询函数，并返回获得的结果。
  * 原函数中的每个return 句都应该像这样：return newQuery\(\)，而不应该返回其他东西。
  * 如果调用者将返回值赋给了一个临时变量，你应该能够去除这个临时 变量。
* 编译，测试。
* 将「原函数的每一个被调用点」替换为「对查询函数的调用」。然后，在调用査询函数的那一行之前，加上对原函数的调用。每次修改后，编译并测试。
* 将原函数的返回值改为void。丨山并删掉其中所有的return 句。

---

## 令函数携带参数 {#_5}

若干函数做了类似的工作，但在函数本体中却包含了不同的值。**建立单一函数，以参数表达那些不同的值。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/10fig05.gif)

**动机（Motivation）**

你可能会发现这样的两个函数：它们做着类似的工作，但因少数几个值致使动作略有不同。这种情况下，你可以将这些各自分离的函数替换为一个统一函数，并通过参数来处理那些变化情况，用以简化问题。这样的修改可以去除重复的代码，并提高灵活性，因为你可以用这个参数处理其他（更多种）变化情况。

**做法（Mechanics）**

* 新建一个带有参数的函数，使它可以替换先前所有的重复性函数（repetitive methods）。
* 编译。
* 将「对旧函数的调用动作」替换为「对新函数的调用动作」。
* 编译，测试。
* 对所有旧函数重复上述步骤，每次替换后，修改并测试。

也许你会发现，你无法用这种办法处理整个函数，但可以处理函数中的一部分代码。 这种情况下，你应该首先将这部分代码提炼到一个独立函数中，然后再对那个提炼所得的函数使用[令函数携带参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_5)。

范例：（Example）

下面是一个最简单的例子：

```
class Employee {
    void tenPercentRaise () {
        salary *= 1.1;
    }

    void fivePercentRaise () {
        salary *= 1.05;
    }
```

这段代码可以替换如下：

```
void raise (double factor) {
    salary *= (1 + factor);
}
```

当然，这个例子实在太简单了，所有人都能做到。

下面是一个稍微复杂的例子：

```
protected Dollars baseCharge() {
    double result = Math.min(lastUsage(),100) * 0.03;
    if (lastUsage() > 100) {
        result += (Math.min (lastUsage(),200) - 100) * 0.05;
    };
    if (lastUsage() > 200) {
        result += (lastUsage() - 200) * 0.07;
    };
    return new Dollars (result);
}
```

上述代码可以替换如下：

```
protected Dollars baseCharge() {
    double result = usageInRange(0, 100) * 0.03;
    result += usageInRange (100,200) * 0.05;
    result += usageInRange (200, Integer.MAX_VALUE) * 0.07;
    return new Dollars (result);
}

protected int usageInRange(int start, int end) {
    if (lastUsage() > start) return Math.min(lastUsage(),end) - start;
    else return 0;
}
```

本项重构的伎俩在于：以「可将少量数值视为参数」为依据，找出带有重复性的代码。

---

## 以明确函数取代参数 {#_13}

你有一个函数，其内完全取决于参数值而采取不同反应。

**针对该参数的每一个可能值，建立一个独立函数。**

```
void setValue (String name, int value) {
    if (name.equals("height"))
        _height = value;
    if (name.equals("width"))
        _width = value;
    Assert.shouldNeverReachHere();
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
void setHeight(int arg) {
    _height = arg;
}
void setWidth (int arg) {
    _width = arg;
}
```

**动机（Motivation）**

[以明确函数取代参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_13)洽恰相反于[令函数携带参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_5)。如果某个参数有离散取值，而函数内又以条件式检查这些参数值，并根据不同参数值做出不同的反应，那么就应该使用本项重构。调用者原本必须赋予参数适当的值，以决定该函数做出何种响应；现在，既然你提供了不同的函数给调用 者使用，就可以避免出现条件式。此外你还可以获得「编译期代码检验」的好处， 而且接口也更清楚。如果以参数值决定函数行为，那么函数用户不但需要观察该函数，而且还要判断参数值是否合法，而「合法的参数值」往往很少在文档中被清楚地提出。

就算不考虑「编译期检验」的好处，只是为了获得一个清晰的接口，也值得你执行本项重构。哪怕只是给一个内部的布尔（boolean）变量赋值，相较之下Switch.beOn\(\) 也比Switch.setState\(true\) 要清楚得-多。

但是，如果参数值不会对函数行为有太多影响，你就不应该使用[以明确函数取代参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_13)。如果情况真是这样，而你也只需要通过参数为一个值域赋值，那么直接使用设值函数（setter）就行了。如果你的确需要「条件判断」 式的行为，可考虑使用[以多态取代条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_6)。

**做法（Mechanics）**

* 针对参数的每一种可能值，新建一个明确函数。
* 修改条件式的每个分支，使其调用合适的新函数。
* 修改每个分支后，编译并测试。
* 修改原函数的每一个被调用点，改而调用上述的某个合适的新函数。
* 编译，测试。
* 所有调用端都修改完毕后，删除原（带有条件判断的）函数。

**范例：（Example）**

下列代码中，我想根据不同的参数值，建立Employee 之下不同的subclass。以下 代码往往是

[以工厂函数取代构造函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_10)

的施行成果：

```
static final int ENGINEER = 0;
static final int SALESMAN = 1;
static final int MANAGER = 2;

static Employee create(int type) {
    switch (type) {
        case ENGINEER:
            return new Engineer();
        case SALESMAN:
            return new Salesman();
        case MANAGER:
            return new Manager();
        default:
            throw new IllegalArgumentException("Incorrect type code value");
    }
}
```

由于这是一个factory method，我不能实施

[以多态取代条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_6)

，因为使用该函数时我根本尚未创建出对象。我并不期待太多新的subclasses，所以一个明确的接口是合理的（译注：不甚理解作者文意）。首先，我要根据参数值建立相应的新函数：

```
static Employee createEngineer() {
    return new Engineer();
}
static Employee createSalesman() {
    return new Salesman();
}
static Employee createManager() {
    return new Manager();
}
```

然后把「switch 语句的各个分支」替换为「对新函数的调用」：

```
static Employee create(int type) {
    switch (type) {
        case ENGINEER:
            return Employee.createEngineer();
        case SALESMAN:
            return new Salesman();
        case MANAGER:
            return new Manager();
        default:
            throw new IllegalArgumentException("Incorrect type code value");
    }
}
```

每修改一个分支，都需要编译并测试，直到所有分支修改完毕为止：

```
static Employee create(int type) {
    switch (type) {
        case ENGINEER:
            return Employee.createEngineer();
        case SALESMAN:
            return Employee.createSalesman();
        case MANAGER:
            return Employee.createManager();
        default:
            throw new IllegalArgumentException("Incorrect type code value");
    }
}
```

接下来，我把注意力转移到旧函数的调用端。我把诸如下面这样的代码：

```
Employee kent = Employee.create(ENGINEER)
```

替换为：

```
Employee kent = Employee.createEngineer()
```

修改完create\(\) 函数的所有调用者之后，我就可以把create\(\) 函数删掉了。同时也可以把所有常量都删掉。

---

## 保持对象完整 {#_6}

你从某个对象中取出若干值，将它们作为某一次函数调用时的参数。

**改使用（传递）整个对象。**

```
int low = daysTempRange().getLow();
int high = daysTempRange().getHigh();
withinPlan = plan.withinRange(low, high);
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
withinPlan = plan.withinRange(daysTempRange());
```

**动机（Motivation）**

有时候，你会将来自同一对象的若干项数据作为参数，传递给某个函数。这样做的问题在于：万一将来被调用函数需要新的数据项，你就必须查找并修改对此函数的所有调用。如果你把这些数据所属的整个对象传给函数，可以避免这种尴尬的处境， 因为被调用函数可以向那个参数对象请求任何它想要的信息。

除了可以使参数列更稳固（不变动）之外，[保持对象完整](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_6)往往还能提高代码的可读性。过长的参数列很难使用，因为调用者和被调用者都必须记住这些参数的用途。此外，不使用完整对象也会造成重复代码，因为被调用函数无法利用完整对象中的函数来计算某些中间值。

「甘蔗不曾两头甜」！如果你传的是数值，被调用函数就只与这些数值有依存关系（dependency），与这些数值所属对象没有任何依存关系。但如果你传递的是整个对象，「参数对象」和「被调用函数所在对象」之间，就有了依存关系。如果这会使你的依存结构恶化，那么你就不该使用[保持对象完整](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_6)。

我还听过另一种不使用[保持对象完整](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_6)的理由：如果被调用函数只需要「参数对象」的其中一项数值，那么只传递那个数值会更好。我并不认同这种观点，因为传递一项数值和传递一个对象，至少在代码清晰度上是等价的〔当然对于pass by value（传值）参数来说，性能上可能有所差异）。更重要的考量应该放在「对象之间的依存关系」上。

如果被调用函数使用了 \[来自另一个对象的很多项数据」，这可能意味该函数实际上应该被定义在「那些数据所属的对象」中。所以，考虑[保持对象完整](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_6)的同时，你也应该考虑[搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)。

运用本项重构之前，你可能还没有定义一个完整对象。那么你就应该先使用[引入参数对象](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_4)。

还有一种常见情况：调用者将自己的若干数据作为参数，传递给被调用函数。这种情况下，如果该对象有合适的取值函数（getter），你可以使用取代这些参数值，并且无须操心对象依存问题。

**做法（Mechanics）**

* 对你的目标函数新添一个参数项，用以代表原数据所在的完整对象。
* 编译，测试。
* 判断哪些参数可被包含在新添的完整对象中。
* 选择上述参数之一，将「被调用函数」内对该参数的各个引用，替换为「对新添之参数对象的相应取值函数（getter）」的调用。
* 删除该项参数。
* 编译，测试。
* 针对所有「可从完整对象中获得」的参数，重复上述过程。
* 删除调用端中那些带有「被删除之参数」的所有代码。
  * 当然，如果调用端还在其他地方使用了这些参数，就不要删除它们。
* 编译，测试。

---

## 以函数取代参数 {#_14}

对象调用某个函数，并将所得结果作为参数，传递给另一个函数。而接受该参数的函数也可以（也有能力）调用前一个函数。

**让参数接受者去除该项参数，并直接调用前一个函数。**

```
int basePrice = _quantity * _itemPrice;
discountLevel = getDiscountLevel();
double finalPrice = discountedPrice (basePrice, discountLevel);
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
int basePrice = _quantity * _itemPrice;
double finalPrice = discountedPrice (basePrice);
```

**动机（Motivation）**

如果函数可以通过其他途径（而非参数列〕获得参数值，那么它就不应该通过参数取得该值。过长的参数列会增加程序阅读者的理解难度，因此我们应该尽可能缩短参数列的长度。

缩减参数列的办法之一就是，看看「参数接受端（receiver）」是否可以通过「与调用端相同的计算」来取得参数携带值。如果调用端通过「其所属对象内部的另一个函数」来计算参数，并在计算过程中「未曾引用调用端的其他参数」（译注：亦就是说没有太多与外界的相依关系），那么你就应该可以将这个计算过程转移到被调用端内，从而去除该项参数。如果你所调用的函数隶属另一对象，而该对象拥有一个reference 指向调用端所属对象，前面所说的这些也同样适用。

但是，如果「参数计算过程」倚赖调用端的某个参数，那么你就无法去掉被调用端的那个参数，因为每一次调用动作中，该参数值都可能不同（当然，如果你能够运用[以明确函数取代参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_13)将该参数替换为一个函数，又另当别论）。另外，如果参数接受端（receiver）并没有一个reference 指向参数发送端（sender），而你也不想加上这样一个reference ，那么也无法去除参数。

有时候，参数的存在是为了将来的弹性。这种情况下我仍然会把这种多余参数拿掉。是的，你应该只在必要关头才添加参数，预先添加的参数很可能并不是你所需要的。不过，对于这条规则，也有一个例外：如果修改接口会对整个程序造成非常痛苦的结果（例如需要很长时间来重建程序，或需要修改大量代码〕，那么可以考虑保留前人预先加入的参数。如果真是这样，你应该首先判断修改接口究竟会造成多严重的后果，然后考虑是否「降低系统各部位之间的依存程度」以减少「修改接口所造成的影响」。稳定的接口确实很好，但是被冻结在一个不良接 口上，也是有问题的。

**做法（Mechanics）**

* 如果有必要，将参数的计算过程提炼到一个独立函数中。
* 将函数本体内「对该参数的引用」替换为「对新建函数的调用」。
* 每次替换后，修改并测试。
* 全部替换完成后，使用
  [移除参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_7)
  将该参数去掉。

---

## 引入参数对象 {#_4}

某些参数总是很自然地同时出现。**以一个对象取代这些参数。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/10fig06.gif)

**动机（Motivation）**

你常会看到特定的一组参数总是一起被传递。可能有好几个函数都使用这一组参数，这些函数可能隶属同一个class，也可能隶属不同的classes 。这样一组参数就是所谓的Date Clump （数据泥团）」，我们可以运用一个对象包装所有这些数据，再以该对象取代它们。哪怕只是为了把这些数据组织在一起，这样做也是值得的。本项重构的价值在于「缩短了参数列的长度」，而你知道，过长的参数列总是难以理解的。此外，新对象所定义的访问函数（accessors）还可以使代码更具一致性，这又进一步降低了代码的理解难度和修改难度。

本项重构还可以带给你更多好处。当你把这些参数组织到一起之后，往往很快可以发现一些「可被移至新建class」的行为。通常，原本使用那些参数的函数对那些参数会有一些共通措施，如果将这些共通行为移到新对象中，你可以减少很多重复代码。

**做法（Mechanics）**

* 新建一个class，用以表现你想替换的一组参数。将这个设为不可变的（不可被修改的，immutable）。
* 编译。
* 针对使用该组参数的所有函数，实施
  [添加参数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_1)
  ，以上述新建class 之实体对象作为新添参数，并将此一参数值设为null 。
  * 如果你所修改的函数被其他很多函数调用，那么你可以保留修改前的旧函数，并令它调用修改后的新函数。你可以先对旧函数进行重构， 然后逐一令调用端转而调用新函数，最后再将旧函数删除。
* 对于Data Clump（数据泥团）中的每一项（在此均为参数），从函数签名式（signature）中移除之，并修改调用端和函数本体，令它们都改而通过「新建 的参数对象」取得该值。
* 每去除一个参数，编译并测试。
* 将原先的参数全部去除之后，观察有无适当函数可以运用
  [搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)
  搬移到参数对象之中。
  * 被搬移的可能是整个函数，也可能是函数中的一个段落。如果是后者， 首先使用
    [提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)
    将该段落提炼为一个独立函数，再搬移这一新建函数。

---

## 移除设值函数 {#_8}

你的class 中的某个值域，应该在对象初创时被设值，然后就不再改变。**去掉该值域的所有设值函数（setter）。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/10fig07.gif)

**动机（Motivation）**

如果你为某个值域提供了设值函数（setter），这就暗示这个值域值可以被改变。如果你不希望在对象初创之后此值域还有机会被改变，那就不要为它提供设值函数 （同时并将该值域设为final ）。这样你的意图会更加清晰，并且往往可以排除其值被修改的可能性——这种可能性往往是非常大的。

如果你保留了间接访问变量的方法，就可能经常有程序员盲目使用它们\[Beck\]。这些人甚至会在构造函数中使用设值函数！我猜想他们或许是为了代码的一致性，但却忽视了设值函数往后可能带来的混淆。

**作法（Mechanics）**

* 检查设值函数（setter）被使用的情况，看它是否只被构造函数调用，或者被构造函数所调用的另一个函数调用。
* 修改构造函数，使其直接访问设值函数所针对的那个变量。
  * 如果某个subclass 通过设值函数给superclass 的某个private 值域设了值，那么你就不能这样修改。这种情况下你应该试着在superclass 中提供一个protected 函数（最好是构造函数）来给这些值域设值。不论你怎么做，都不要给superclass 中的函数起一个与设值函数混淆的名字。
* 编译，测试。
* 移除这个设值函数，将它所计对的值域设为final 。
* 编译，测试。

---

## 隐藏函数 {#_3}

有一个函数，从来没有被其他任何class 用到。**将这个函数修改为private 。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/10fig08.gif)

**动机（Motivation）**

重构往往促使你修改「函数的可见度」（ visibility of methods）。提高函数可见度的情况很容易想像：另一个class 需要用到某个函数，因此你必须提高该函数的可见度。但是要指出一个函数的可见度是否过高，就稍微困难一些。理想状况下你可以使用工具检查所有函数，指出可被隐藏起来的函数。即使没有这样的工具，你也应该时常进行这样的检查。

一种特别常见的情况是：当你而对一个过于丰富、提供了过多行为的接口时，就值得将非必要的取值函数（getter）和设值函数（setter）隐藏起来。尤其当你面对的是一个「只不过做了点简单封装」的数据容器（data holder）时，情况更是如此。 随着愈来愈多行为被放入这个class 之中，你会发现许多取值/设值函数不再需要为public ，因此可以把它们隐藏起来。如果你把取值/设值函数设为private ，并在他处直接访问变量，那就可以放心移除取值/设值函数了。

**作法（Mechanics）**

* 经常检查有没有可能降低某个函数的可见度（使它更私有化）。
  * 使用lint-style 工具，尽可能频繁地检查。当你在另一个class 中移除对某个函数的调用时，也应该进行检查。
  * 特别对设值函数（setter）进行上述的检查。
* 尽可能降低所有函数的可见度。
* 每完成一组函数的隐藏之后，编译并测试。
  * 如果有不适当的隐藏，编译器很自然会检验出来，因此不必每次修改 后都进行编译。如有任何错误出现，很容易被发现。

---

## 以\[工厂函数\]取代\[构造函数\] {#_10}

你希望在创建对象时不仅仅是对它做简单的建构动作（simple construction ）。

**将constructor （构造函数）替换为factory method（工厂函数）。**

```
Employee (int type) {
    _type = type;
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
static Employee create(int type) {
    return new Employee(type);
}
```

**动机（Motivation）**

使用[以工厂函数取代构造函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_10)的最显而易见的动机就是在subclassing 过程中以factory method 以取代type code。你可能常常需要根据type code 创建相应的对象，现在，创建名单中还得加上subclasses，那些subclasses 也是根据type code 来创建。然而由于构造函数只能返回「被索求之对象」，因此你需要将构造函数替换为Factory Method \[Gang of Four\]。

此外，如果构造函数的功能不能满足你的需要，也可以使用factory method 来代替它。Factory method 也是[将实值对象改为引用对象](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_4)的基础。你也可以令你的factory method 根据参数的个数和型别，选择不同的创建行为。

**做法（Mechanics）**

* 新建一个factory method ，让它调用现有的构造函数。
* 将「对构造函数的调用」替换为「对factory method 的调用」。
* 每次替换后，编译并测试。
* 将构造函数声明为private。
* 编译。

## 封装\[向下转型\]动作 {#_2}

## 以异常取代错误码 {#_11}

## 以测试取代异常 {#_12}




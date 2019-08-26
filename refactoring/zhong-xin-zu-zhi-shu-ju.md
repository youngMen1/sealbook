## 自封装字段

你直接访问一个值域（field），但与值域之间的耦合关系逐渐变得笨拙。

为这个值域建立取值/设值函数（getting and setting methods ），并且只以这些函数来访问值域。

```
private int _low, _high;
boolean includes (int arg) {
    return arg >= _low && arg <= _high;
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
private int _low, _high;
boolean includes (int arg) {
    return arg >= getLow() && arg <= getHigh();
}
int getLow() {return _low;}
int getHigh() {return _high;}
```

**动机（Motivation）**

在「值域访问方式」这个问题上，存在两种截然不同的观点：其中一派认为，在该 变量定义所在的中，你应该自由（直接）访问它；另一派认为，即使在这个class中你也应该只使用访问函数间接访问之。两派之间的争论可以说是如火如荼。（参见Auer在\[Auer\]p.413和Beck在\[Beck\]上的讨论。）

本质而言，「间接访问变量」的好处是，subclass得以通过「覆写一个函数」而改变获取数据的途径；它还支持更灵活的数据管理方式，例如lazy initialization \(意思是: 只有在需要用到某值时，才对它初始化）。

「直接访问变量」的好处则是：代码比较容易阅读。阅读代码的时候，你不需要停下来说：『啊，这只是个取值函数。』

面临选择时，我总是做两手准备。通常情况下我会很乐意按照团队中其他人的意愿来做。就我自己而言，我比较喜欢先使用「直接访问」方式，直到这种方式给我带来麻烦为止。如果「直接访问」给我带来麻烦，我就会转而使用「间接访问」方式。 重构给了我改变主意的自由。

如果你想访问superclass中的一个值域，却又想在subclass中将「对这个变量的访问」改为一个计算后的值，这就是最该使用[封装值域](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_7)的时候。「值域自我封装」只是第一步。完成自我封装之后，你可以在subclass中根据自己的需要随意覆写取值/设值函数（getting and setting methods ）。

**做法（Mechanics）**

* 为「待封装值域」建立取值/设值函数（getting and setting methods）。
* 找出该值域的所有引用点，将它们全部替换为「对于取值/设值函数的调用」。
* 如果引用点是「读取」值域值，就将它替换为「调用取值函数」；如果引用点是「设定」值域值，就将它替换为「调用设值函数」。
* 你可以暂时为该值域改名，让编译器帮助你查找引用点。
* 将该值域声明为private。
* 复查，确保找出所有引用点。
* 编译，测试。

**范例（Example）**

下面这个例子看上去有点过分简单。不过，嘿，起码它写起来很快：

```
class IntRange {

    private int _low, _high;

    boolean includes (int arg) {
        return arg >= _low && arg <= _high;
    }

    void grow(int factor) {
        _high = _high * factor;
    }
    IntRange (int low, int high) {
        _low = low;
        _high = high;
    }
```

为了封装\_low和\_high这两个值域，我先定义「取值/设值函数」（如果此前没有定义的话），并使用它们：

```
class IntRange {

    boolean includes (int arg) {
        return arg >= getLow() && arg <= getHigh();
    }

    void grow(int factor) {
        setHigh (getHigh() * factor);
    }

    private int _low, _high;

    int getLow() {
       return _low;
    }

    int getHigh() {
        return _high;
    }

    void setLow(int arg) {
        _low = arg;
    }

    void setHigh(int arg) {
        _high = arg;
    }
```

使用本项重构时，你必须小心对待「在构造函数中使用设值函数」的情况。一般说来，设值函数被认为应该在「对象创建后」才使用，所以初始化过程中的行为有可能与设值函数的行为不同。在这种情况下，我也许在构造函数中直接访问值域，要不就是建立另一个独立的初始化函数：

```
IntRange (int low, int high) {
    initialize (low, high);
}

private void initialize (int low, int high) {
    _low = low;
    _high = high;
}
```

一旦你拥有一个subclass，上述所有动作的价值就体现出来了。如下所示：

```
class CappedRange extends IntRange {

    CappedRange (int low, int high, int cap) {
        super (low, high);
        _cap = cap;
    }

    private int _cap;

    int getCap() {
        return _cap;
    }

    int getHigh() {
        return Math.min(super.getHigh(), getCap());
    }
}
```

现在，我可以在CappedRange class中覆写getHigh\(\)，从而加入对cap的考虑，而不必修改IntRange class的任何行为。

---

## 以对象取代数据值

你有一笔数据项（data item），需要额外的数据和行为。

**将这笔数据项变成一个对象。**

![](/assets/微信截图_20190727085505.png)

**动机（Motivation）**

开发初期，你往往决定以简单的数据项（data item）表示简单的行为。但是，随着开发的进行，你可能会发现，这些简单数据项不再那么简单了。比如说，一开始你可能会用一个字符串来表示「电话号码」概念，但是随后你就会发现，电话号码需要「格式化」、「抽取区号」之类的特殊行为。如果这样的数据项只有一二个，你还可以把相关函数放进数据项所属的对象里头；但是Duplication Code臭味和Feature Envy臭味很快就会从代码中散发出来。当这些臭味开始出现，你就应该将数据值（data value）变成对象（object）。

**做法**（Mechanics）

* 为「待替换数值」新建一个class，在其中声明一个final值域，其型别和source class中的「待替换数值」型别一样。然后在新class中加入这个值域的取值函数（getter），再加上一个「接受此值域为参数」的构造函数。
* 编译。
* 将source class中的「待替换数值值域」的型别改为上述的新建class。
* 修改source class中此一值域的取值函数（getter），令它调用新建class的取值函数。
* 如果source class构造函数中提及这个「待替换值域」（多半是赋值动作），我们就修改构造函数，令它改用新的构造函数来对值域进行赋值动作。
* 修改source class中「待替换值域」的设值函数（setter），令它为新class创建一个实体。
* 编译，测试。
* 现在，你有可能需要对新class使用[将实值对象改为引用对象](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_4)。

**范例（Example）**

下面有一个代表「定单」的Order class，其中以一个字符串记录定单客户。现在，我希望改以一个对象来表示客户信息，这样我就有充裕的弹性保存客户地址、信用 等级等等信息，也得以安置这些信息的操作行为。Order class最初如下：

```
class Order...
    public Order (String customer) {
        _customer = customer;
    }
    public String getCustomer() {
        return _customer;
    }
    public void setCustomer(String arg) {
        _customer = arg;
    }
    private String _customer;
```

Order class的客户代码可能像下面这样：

```
private static int numberOfOrdersFor(Collection orders, String customer) {
    int result = 0;
    Iterator iter = orders.iterator();
    while (iter.hasNext()) {
        Order each = (Order) iter.next();
        if (each.getCustomerName().equals(customer)) result++;
    }
    return result;
}
```

首先，我要新建一个Customer class来表示「客户」概念。然后在这个class中建立一个final值域，用以保存一个字符串，这是Order class目前所使用的。我将这个新值域命名为\_name，因为这个字符串的用途就是记录客户名称。此外我还要为这个字符串加上取值函数（getter）和构造函数（constructor）。

```
class Customer {
    public Customer (String name) {
        _name = name;
    }
    public String getName() {
        return _name;
    }
    private final String _name;
}
```

现在，我要将Order中的\_customer值域的型别修改为Customer；并修改所有引用此一值域的函数，让它们恰当地改而使用Customer实体。其中取值函数和构造函数的修改都很简单；至于设值函数（setter），我让它创建一份Customer实体。

```
class Order...
    public Order (String customer) {
        _customer = new Customer(customer);
    }
    public String getCustomer() {
        return _customer.getName();
    }
    private Customer _customer;

    public void setCustomer(String arg) {
        _customer = new Customer(customer);
    }
```

---

## 将值对象改为引用对象

你有一个class，衍生出许多相等实体（equal instances），你希望将它们替换为单一对象。

**将这个value object （实值对象）变成一个reference object \(引用对象）。**

**动机（Motivation）**

在许多系统中，你都可以对对象做一个有用的分类：reference object和value objects。前者就像「客户」、「帐户」这样的东西，每个对象都代表真实世界中的一个实物，你可以直接以相等操作符（==，用来检验同一性，identity）检査两个对象 是否相等。后者则是像「日期」、「钱」这样的东西，它们完全由其所含的数据值来定义，你并不在意副本的存在；系统中或许存在成百上千个内容为"1/1/2000"的「日期」对象。当然，你也需要知道两个value objects是否相等，所以你需要覆写equals\(\)和hashCode\(\)）。

要在reference object和value objects之间做选择有时并不容易。有时候，你会从一个简单的value objects开始，在其中保存少量不可修改的数据。而后，你可能会希望给这个对象加入一些可修改数据，并确保对任何一个对象的修改都能影响到所有引用此一对象的地方。这时候你就需要将这个对象变成一个reference object。

**做法（Mechanics）**

* 使用
  [以工厂函数取代构造函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_10)
  。
* 编译，测试。
* 决定由什么对象负责提供访问新对象的途径。
* 可能是个静态字典（static dictionary）或一个注册对象（registry object）。
* 你也可以使用多个对象作为新对象的访问点（access point）。
* 决定这些reference object应该预先创建好，或是应该动态创建。
  * 如果这些reference object是预先创建好的,而你必须从内存中将它们读取出来，那么就得确保它们在被需要的时候能够被及时加载。
  * 修改factory method 5 ，令它返回reference object。
  * 如果对象是预先创建好的，你就需要考虑：万一有人索求一个其实并不存在的对象，要如何处理错误？
  * 你可能希望对factory method使用
    [重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)
    ，使其传达这样的信息：它返回的是一个既存对象。
* 编译，测试。

5译注：此处之factory method不等同于GoF在《Design Patterns》书中提出的Factory Method。为避免混淆，读者应该将此处的factory method理解为"Creational Method"，亦即「用以创建某种实体」的函数，这个概念包含GoF的Factory Method，而又比Factory Method广泛。

**范例（Example）**

在[以对象取代数据值](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_9)一节中，我留下了一个重构后的程序，本节范例就从它开始。我们有下列的Customer class：

```
class Customer {
    public Customer (String name) {
        _name = name;
    }
    public String getName() {
        return _name;
    }
    private final String _name;
}
```

它被以下的Ordre class使用：

```
class Order...
    public Order (String customerName) {
        _customer = new Customer(customerName);
    }
    public void setCustomer(String customerName) {
        _customer = new Customer(customerName);
    }
    public String getCustomerName() {
        return _customer.getName();
    }
    private Customer _customer;
```

此外，还有一些代码也会使用Customer对象：

```
private static int numberOfOrdersFor(Collection orders, String customer) {
    int result = 0;
    Iterator iter = orders.iterator();
    while (iter.hasNext()) {
        Order each = (Order) iter.next();
        if (each.getCustomerName().equals(customer)) result++;
    }
    return result;
}
```

---

## 将引用对象改为值对象

你有一个reference object（引用对象），很小且不可变（immutable），而且不易管理。

将它变成一个value object（实值对象）。![](http://wangvsa.github.io/refactoring-cheat-sheet/images/08fig03.gif)

**动机（Motivation）**

正如我在[将引用对象改为实值对象](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_2)中所说，要在reference object和value object之间做选择，有时并不容易。作出选择后，你常会需要一条回头路。

如果reference object开始变得难以使用，也许你就应该将它改为value object。 reference object必须被某种方式控制，你总是必须向其控制者请求适当的reference object。它们可能造成内存区域之间错综复杂的关联。在分布系统和并发系统中，不可变的value object特别有用，因为你无须考虑它们的同步问题。

value object有一个非常重要的特性：它们应该是不可变的（immutable）。无论何时只要你调用同一对象的同一个查询函数，你都应该得到同样结果。如果保证了这一 点，就可以放心地以多个对象表示相同事物（same thing）。如果value object是可变的（mutable），你就必须确保你对某一对象的修改会自动更新其他「代表相同事物」的其他对象。这太痛苦了，与其如此还不如把它变成reference object。

这里有必要澄清一下「不可变（immutable）」的意思。如果你以Money class表示「钱」的概念，其中有「币种」和「金额」两条信息，那么Money对象通常是一个不可变的value object。这并非意味你的薪资不能改变，而是意味：如果要改变你的薪资，你需要使用另一个崭新的Money对象来取代现有的Money对象，而不是在现有的Money对象上修改。你和Money对象之间的关系可以改变，但Money对象自身不能 改变。

译注：《Practical Java》 by Peter Haggar第6章对于mutable/immutable有深入讨论。

**做法（Mechanics）**

* 检查重构对象是否为immutable（不可变）对象，或是否可修改为不可变对象。
* 如果该对象目前还不是immutable，就使用
  [移除设置函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_8)
  ，直到它成为immutable为止。
* 如果无法将该对象修改为immutable，就放弃使用本项重构。
* 建立equal\(\)和hashCode\(\)。
* 编译，测试。
* 考虑是否可以删除factory method，并将构造函数声明public 。

**范例（Example）**

让我们从一个表示「货币种类」的Currency class开始：

```
class Currency...
    private String _code;

    public String getCode() {
       return _code;
    }
    private Currency (String code) {
       _code = code;
    }
```

这个也所做的就是保存并返回一个货币种类代码。它是一个reference object，所以如果要得到它的一份实体，必须这么做：

```
Currency usd = Currency.get("USD");
```

Currency class维护一个实体链表（list of instances）；我不能直接使用构造函数创建实体，因为Currency构造函数是private。

---

## 以对象取代数组

你有一个数组（array），其中的元素各自代表不同的东西。

以对象替换数组。对于数组中的每个元素，以一个值域表示之。

```
String[] row = new String[3];
row [0] = "Liverpool";
row [1] = "15";
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
Performance row = new Performance();
row.setName("Liverpool");
row.setWins("15");
```

**动机（Motivation）**

数组（array）是一种常见的用以组织数据的结构体。不过，它们应该只用于「以某种顺序容纳一组相似对象」。有时候你会发现，一个数组容纳了数种不同对象，这会给array用户带来麻烦，因为他们很难记住像「数组的第一个元素是人名」这样的约定。对象就不同了，你可以运用值域名称和函数名称来传达这样的信息，因此你无需死记它，也无需倚赖注释。而且如果使用对象，你还可以将信息封装起来，并使用[搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)为它加上相关行为。

**做法（Mechanics）**

* 新建一个class表示数组所示信息，并在该class中以一个public值域保存原先的数组。
* 修改数组的所有用户，让它们改用新建的class实体。
* 编译，测试。
* 逐一为数组元素添加取值/设值函数（getters/setters）。根据元素的用途，为这些访问函数命名。修改客户端代码，让它们通过访问函数取用数组内的元素。 每次修改后，编译并测试。
* 当所有「对数组的直接访问」都被取代为「对访问函数的调用」后，将class之中保存该数组的值域声明private。
* 编译。
* 对于数组内的每一个元素，在新class中创建一个型别相当的值域；修改该元素的访问函数，令它改用上述的新建值域。
* 每修改一个元素，编译并测试。
* 数组的所有元素都在对应的class内有了相应值域之后，删除该数组。

---

## 复制“被监控数据”

\(译注：本节大量保留domain，presentation，event，getter/setter，observed等字眼。所谓presentation class，用以处理「数据表现形式」；所谓domain class，用以处理业务逻辑。）

你有一些domain class置身于GUI控件中，而domain method需要访问之。

将该笔数据拷贝到一个domain object中。建立一个Observer模式，用以对domain object和GUI object内的重复数据进行同步控制（sync.）。

![](/assets/微信截图_20190819201141.png)

**动机（Motivation）**

一个分层良好的系统，应该将处理用户界面（UI）和处理业务逻辑（business logic）的代码分开。之所以这样做，原因有以下几点：\(1\) 你可能需要使用数个不同的用 户界面来表现相同的业务逻辑；如果同时承担两种责任，用户界面会变得过分复杂； \(2\) 与GUI隔离之后，domain class的维护和演化都会更容易；你甚至可以让不同的开发者负责不同部分的开发。

尽管你可以轻松地将「行为」划分到不同部位，「数据」却往往不能如此。同一笔 数据有可能既需要内嵌于GUI控件，也需要保存于domain model里头。自从MVC（Model-View-Controller）模式出现后，用户界面框架都使用多层系统（multitiered system）来提供某种机制，使你不但可以提供这类数据，并保持它们同步（sync.）。

如果你遇到的代码是以双层（two-tiered）方式开发，业务逻辑（business logic）被内嵌于用户界面（UI）之中，你就有必要将行为分离出来。其中的主要工作就是函数的分解和搬移。但数据就不同了：你不能仅仅只是移动数据，你必须将它复制到新建部位中，并提供相应的同步机制。

## 将单向关联改为双向关联

两个classes都需要使用对方特性，但其间只有一条单向连接（one-way link）。

添加一个反向指针，并使修改函数（modifiers）能够同时更新两条连接。（译注：这里的指针等同于句柄（handle），修改函数（modifier）指的是改变双方关系者）![](http://wangvsa.github.io/refactoring-cheat-sheet/images/08fig06.gif)

**动机（Motivation）**

开发初期，你可能会在两个classes之间建立一条单向连接，使其中一个可以引用另一个class。随着时间推移，你可能发现referred class 需要得到其引用者（某个object）以便进行某些处理。也就是说它需要一个反向指针。但指针乃是一种单向连接，你不可能反向操作它。通常你可以绕道而行，虽然会耗费一些计算时间， 成本还算合理，然后你可以在referred class中建立一个专职函数，负责此一行为。 但是，有时候，想绕过这个问题并不容易，此时你就需要建立双向引用关系（two-way reference），或称为反向指针（back pointer）。如果你不习惯使用反向指针，它们很容易造成混乱；但只要你习惯了这种手法，它们其实并不是太复杂。

「反向指针」手法有点棘手，所以在你能够自在运用它之前，应该有相应的测试。通常我不花心思去测试访问函数（accessors），因为普通访问函数的风险没有高到需要测试的地步，但本重构要求测试访问函数，所以它是极少数需要添加测试的重构 手法之一。

本重构运用反向指针（back pointer）实现双向关联（bidirectionality）。其他技术（例如连接对象，link object）需要其他重构手法。

**做法（Mechanics）**

* 在class中增加一个值域，用以保存「反向指针」。
* 决定由哪个class \(引用端或被引用端）控制关联性（association）。
* 在「被控端」建立一个辅助函数，其命名应该清楚指出它的有限用途。
* 如果既有的修改函数（modifier）在「控制端」，让它负责更新反向指针。
* 如果既有的修改函数（modifier）在「被控端」，就在「控制端」建立一个控制函数，并让既有的修改函数调用这个新建的控制函数。

---

## 将双向关联改为单向关联

---

## 以字面常量取代魔法数

---

## 封装字段

---

## 封装集合

有个函数（method）返回一个群集（collection）。

让这个函数返回该群集的一个只读映件（read-only view），并在这个class中提供「添加/移除」（add/remove）群集元素的函数。 !\[\]\(../images/08fig08.gif"/&gt;

**动机（Motivation）**

class常常会使用群集（collection，可能是array、list、set或vector）来保存一组实体。这样的class通常也会提供针对该群集的「取值/设值函数」（getter/setter）。

但是，群集的处理方式应该和其他种类的数据略有不同。取值函数（getter）不该返回群集自身，因为这将让用户得以修改群集内容而群集拥有者却一无所悉。这也会对用户暴露过多「对象内部数据结构」的信息。如果一个取值函数（getter）确实需要返回多个值，它应该避免用户直接操作对象内所保存的群集，并隐藏对象内「与用户无关」的数据结构。至于如何做到这一点，视你使用的版本不同而有所不同。

另外，不应该为这整个群集提供一个设值函数（setter），但应该提供用以为群集添加/移除（add/remove）元素的函数。这样，群集拥有者（对象）就可以控制群集元素的添加和移除。

如果你做到以上数点，群集（collection）就被很好地封装起来了，这便可以降低群集拥有者（class）和用户之间的耦合度。

做**法（Mechanics）**

* 加入「为群集添加（add）、移除（remove）元素」的函数。
* 将「用以保存群集」的值域初始化为一个空群集。
* 编译。
* 找出「群集设值函数」的所有调用者。你可以修改那个设值函数，让它使用 上述新建立的「添加/移除元素」函数；也可以直接修改调用端，改让它们调用上述新建立的「添加/移除元素」函数。
  * 两种情况下需要用到「群集设值函数」：\(1\) 群集为空时；\(2\) 准备将原有群集替换为另一个群集时。
  * 你或许会想运用
    [重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)
    为「群集设值函数」改名，从setXxx\(\)改为initialzeXxx\(\)或replaceXxx\(\)。
* 编译，测试。
* 找出所有「通过取值函数（getter）获得群集并修改其内容」的函数。逐一修改这些函数，让它们改用「添加/移除」（add/remove）函数。每次修改后，编译并测试。
* 修改完上述所有「通过取值函数（getter）获得群集并修改群集内容」的函数后，修改取值函数自身，使它返回该群集的一个只读映件（read-only view）。
  * 在Java 2中，你可以使用Collection.unmodifiableXxx\(\)得到该群集的只读映件。
  * 在Java 1.1中，你应该返回群集的一份拷贝。
* 编译，测试。
* 找出取值函数（getter）的所有用户，从中找出应该存在于「群集之宿主对象（host object）内的代码。运用
  [提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)
  和
  [搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)
  将这些代码移到宿主对象去。

如果你使用Java 2，那么本项重构到此为止。如果你使用Java 1.1，那么用户也许会喜欢使用枚举（enumeration）。为了提供这个枚举，你应该这样做：

* 修改现有取值函数（getter）的名字，然后添加一个新取值函数，使其返回一个枚举。找出旧取值函数的所有被使用点，将它们都改为使用新取值函数。
  * 如果这一步跨度太大，你可以先使用
    [重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)
    修改原取值函数的名称；再建立一个新取值函数用以返回枚举；最后再修改 所有调用者，使其调用新取值函数。
* 编译，测试。

---

## 以数据类取代记录

你需要面对传统编程环境中的record structure （记录结构）。

_为该record （记录）创建一个「咂」数据对象（dumb data object）。_

**动机（Motivation）**

Record structures （记录型结构）是许多编程环境的共同性质。有一些理由使它们被带进面向对象程序之中：你可能面对的是一个老旧程序（ legacy program ），也可能需要通过一个传统API 来与structured record 交流，或是处理从数据库读出的 records。这些时候你就有必要创建一个interfacing class ，用以处理这些外来数据。最简单的作法就是先建立一个看起来类似外部记录（external record）的class ，以便日后将某些值域和函数搬移到这个class 之中。一个不太常见但非常令人注目的情况是：数组中的每个位置上的元素都有特定含义，这种情况下你应该使用[以对象取代数组](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_8)。

**作法（Mechanics）**

* 新建一个class ，表示这个record 。
* 对于record 中的每一笔数据项，在新建的class 中建立对应的一个private 值域， 并提供相应的取值丨设值函数（getter/setter）。

现在，你拥有了一个「哑」数据对象（dumb data object）。这个对象现在还没有任何有用行为（函数〕，但是更进一步的重构会解决这个问题。

---

## 以类取代类型码

class 之中有一个数值型别码（ numeric type code ），但它并不影响class 的行为。

_以一个新的class 替换该数值型别码（type code）。_

!\[\]\(../images/08fig09.gif"/&gt;

**动机（Motivation）**

在以C 为基础的编程语言中，type code（型别码）或枚举值（enumerations）很常见。如果带着一个有意义的符号名，type code 的可读性还是不错的。问题在于，符号名终究只是个别名，编译器看见的、进行型别检验的，还是背后那个数值。任何接受type code 作为引数（argument）的函数，所期望的实际上是一个数值，无法强制使用符号名。这会大大降低代码的可读性，从而成为臭虫之源。

如果把那样的数值换成一个class ，编译器就可以对这个class 进行型别检验。只要为这个class 提供factory methods ，你就可以始终保证只有合法的实体才会被创建出 来，而且它们都会被传递给正确的宿主对象。

但是，在使用[以类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_13)之前，你应该先考虑type code 的其他替换方式。只有当type code 是纯粹数据时（也就是type code 不会在switch 语句中引起行为变化时），你才能以class 来取代它。Java 只能以整数作为switch 语句的「转辙」依据，不能使用任意class ，因此那种情况下不能够以class 替换type code 。更重要的是：任何switch 语句都应该运用[以多态取代条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_6)去掉。为了进行那样的重构，你首先必须运用[以子类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_14)或[以State/Strategy取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#statestrategy)把type code处理掉。

即使一个type code 不会因其数值的不同而引起行为上的差异，宿主类中的某些行为还是有可能更适合置放于type code class 中，因此你还应该留意是否有必要使用[搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)将一两个函数搬过去。

**做法（Mechanics）**

* 为type code 建立一个class 。
  * 这个class 内需要一个用以记录type code 的值域，其型别应该和type code 相同；并应该有对应的取值函数（getter）。此外还应该用一组static 变量保存「允许被创建」的实体，并以一个对static 函数根据原本的type code 返回合适的实体。
* 修改source class 实现码，让它使用上述新建的class 。
  * 维持原先以type code 为基础的函数接口，但改变static 值域，以新建的class 产生代码。然后，修改type code 相关函数，让它们也从新建的class 中获取代码。
* 编译，测试。
  * 此时，新建的class 可以对type code 进行运行期检查。
* 对于source class 中每一个使用type code 的函数，相应建立一个函数，让新函数使用新建的class 。
  * 你需要建立「以新class 实体为自变量」的函数，用以替换原先「直接以type code 为引数」的函数。你还需要建立一个「返回新class 实体」的函数，用以替换原先「直接返回type code」的函数。建立新函数前，你可以使用
    [重新命名函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_9)
    修改原函数名称，明确指出那些函数仍然使用旧式的type code ，这往往是个明智之举。
* 逐一修改source class 用户，让它们使用新接口。
* 每修改一个用户，编译并测试。
  * 你也可能需要一次性修改多个彼此相关的函数，才能保持这些函数之 间的一致性，才能顺利地编译、测试。
* 删除「使用type code」的旧接口，并删除「保存旧type code」的静态变量。
* 编译，测试。

---

## 以子类取代类型码

你有一个不可变的（immutable）type code ，它会影响class 的行为。

_以一个subclass 取代这个type code。_![](http://wangvsa.github.io/refactoring-cheat-sheet/images/08fig10.gif)

**动机（Motivation）**

如果你面对的type code 不会影响宿主类的行为，你可以使用[以类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_13)来处理它们。但如果type code 会影响宿主类的行为，那么最好的办法就是借助多态（polymorphism ）来处理变化行为。

一般来说，这种情况的标志就是像switch 这样的条件式。这种条件式可能有两种表现形式：switch 语句或者if-then-else 结构。不论哪种形式，它们都是检查type code 值，并根据不同的值执行不同的动作。这种情况下你应该以[以多态取代条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_6)进行重构。但为了能够顺利进行那样的重构，首先应该将type code 替换为可拥有多态行为的继承体系。这样的一个继承体系应该以type code 的宿主类为base class，并针对每一种type code 各建立一个subclass 。

为建立这样的继承体系，最简单的办法就是[以子类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_14)：以type code 的宿主类为base class，针对每种type code 建立相应的subclass 。 但是以下两种情况你不能那么做：\(1\) type code 值在对象创建之后发生了改变；\(2\) 由于某些原因，type code 宿主类已经有了subclass 。如果你恰好面临这两种情况之一，就需要使用[以State/Strategy取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#statestrategy)。

[以子类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_14)的主要作用其实是搭建一个舞台，让[以多态取代条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_6)得以一展身手。如果宿主类中并没有出现条件式，那么[以类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_13)更合适，风险也比较低。使用[以子类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_14)的另一个原因就是，宿主类中出现 了「只与具备特定type code 之对象相关」的特性。完成本项重构之后，你可以使用[函数下移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_10)和[值域下移](http://wangvsa.github.io/refactoring-cheat-sheet/dealing-with-generalization/#_9)将这些特性推到合适的subclass去，以彰显它们「只与特定情况相关」这一事实。

[以子类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_14)的好处在于：它把「对不同行为的了解」从class 用户那儿转移到了class 自身。如果需要再加入新的行为变化，我只需添加subclass 一个就行了。如果没有多态机制，我就必须找到所有条件式，并逐一修改它们。因此，如果未来还有可能加入新行为，这项重构将特别有价值。

**作法（Mechanics）**

* 使用Self-encapsulate Field 将type code 自我封装起来。
* 如果type code 被传递给构造函数，你就需要将构造函数换成factory method。
* 为type code 的每一个数值建立一个相应的subclass 。在每个subclass 中覆写（override）type code的取值函数（getter），使其返回相应的type code 值。
* 这个值被硬编码于return 中（例如：return 1）。这看起来很骯脏， 但只是权宜之计。当所有case 子句都被替换后，问题就解决了。
* 每建立一个新的subclass ，编译并测试。
* 从superclass 中删掉保存type code 的值域。将type code 访问函数（accessors）声明为抽象函数（abstract method）。
* 编译，测试。

---

## 以state/Strategy取代类型码

你有一个type code ，它会影响class 的行为，但你无法使用subclassing。

_以state object （专门用来描述状态的对象）取代type code 。_![](http://wangvsa.github.io/refactoring-cheat-sheet/images/08fig11.gif)

**动机（Motivation）**

本项重构和[以子类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_14)很相似，但如果「type code 的值在对象生命期中发生变化」或「其他原因使得宿主类不能被subclassing 」，你也可以使用本重构。本重构使用State 模式或Stategy 模式\[Gang of Four\]。

State 模式和Stategy 模式非常相似，因此无论你选择其中哪一个，重构过程都是相同的。「选择哪一个模式」并非问题关键所在，你只需要选择更适合特定情境的模式就行了。如果你打算在完成本项重构之后再以[以多态取代条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_6)简化一个算法，那么选择Stategy 模式比较合适；如果你打算搬移与状态相关（state-specific）的数据，而且你把新建对象视为一种变迁状态 （changing state），就应该选择使用State 模式。

**作法（Mechanics）**

* 使用Self-encapsulate Field 将type code 自我封装起来。
* 新建一个class ，根据type code 的用途为它命名。这就是一个state object。
* 为这个新建的class 添加subclass ，每个subclass 对应一种type code 。
  * 比起逐一添加，一次性加入所有必要的subclass 可能更简单些。
* 在superclass 中建立一个抽象的查询函数（abstract query ），用以返回type code 。 在每个subclass 中覆写该函数，返回确切的type code 。
* 编译。
* 在source class 中建立一个值域，用以保存新建的state object。
* 调整source class 中负责查询type code 的函数，将查询动作转发给state object 。
* 调整source class 中「为type code 设值」的函数，将一个恰当的state object subclass 赋值给「保存state object」的那个值域。
* 编译，测试。

---

## 以字段取代子类

你的各个subclasses 的惟一差别只在「返回常量数据」的函数身上。

_修改这些函数，使它们返回superclass 中的某个（新增）值域，然后销毁subclasses 。_![](http://wangvsa.github.io/refactoring-cheat-sheet/images/08fig12.gif)

**动机（Motivation）**

建立subclass 的目的，是为了增如新特性，或变化其行为。有一种变化行为（variant behavior ）称为「常量函数」（constant method）\[Beck\]，它们会返回一个硬编码 （hard-coded）值。这东西有其用途：你可以让不同的subclasses 中的同一个访问函数（accessors）返回不同的值。你可以在superclass 中将访问函数声明为抽象函数， 并在不同的subclass 中让它返回不同的值。

尽管常量函数有其用途，但若subclass 中只有常量函数，实在没有足够的存在价值。 你可以在中设计一个与「常量函数返回值」相应的值域，从而完全去除这样的subclass 。如此一来就可以避免因subclassing 而带来的额外复杂性。

**作法（Mechanics）**

* 对所有subclasses 使用
  [以工厂函数取代构造函数](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_10)
  。
* 如果有任何代码直接引用subclass，令它改而引用superclass 。
* 针对每个常量函数，在superclass 中声明一个final 值域。
* 为superclass 声明一个protected 构造函数，用以初始化这些新增值域。
* 新建或修改subclass 构造函数，使它调用superclass 的新增构造函数。
* 编译，测试。
* 在superclass 中实现所有常量函数，令它们返回相应值域值，然后将该函数从subclass 中删掉。
* 每删除一个常量函数，编译并测试。
* subclass 中所有的常量函数都被删除后，使用
  [将函数内联化](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_2)
  将subclass 构造函数内联（inlining）到superclass 的factory method 中。
* 编译，测试。
* 将subclass 删掉。
* 编译，测试。
* 重复「inlining 构造函数、删除subclass」过程，直到所有subclass 都被删除。




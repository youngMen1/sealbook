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

将这笔数据项变成一个对象。

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


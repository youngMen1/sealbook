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




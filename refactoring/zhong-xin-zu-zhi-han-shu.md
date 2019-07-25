## 提炼函数

你有一段代码可以被组织在一起并独立出来。**将这段代码放进一个独立函数中，并让函数名称解释该函数的用途。**

```
void printOwing(double amount) {
    printBanner();
    //print details
    System.out.println ("name:" + _name);
    System.out.println ("amount" + amount);
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
void printOwing(double amount) {
    printBanner();
    printDetails(amount);
}
void printDetails (double amount) {
    System.out.println ("name:" + _name);
    System.out.println ("amount" + amount);
}
```

1.无局部变量

2.有局部变量

局部变量最简单的情况是：被提炼码只是读取这些变量的值，并不修改它们。这种情况下我可以简单地将它们当作参数传给目标函数。

**动机**

[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)是我最常用的重构手法之一。当我看见一个过长的函数或者一段需要注释才能让人理解用途的代码，我就会将这段代码放进一个独立函数中。

有数个原因造成我喜欢简短而有良好命名的函数。首先，如果每个函数的粒度都很小（finely grained），那么函数之间彼此复用的机会就更大；其次，这会使高层函数码读起来就像一系列注释；再者，如果函数都是细粒度，那么函数的覆写（overridden）也会更容易些。

的确，如果你习惯看大型函数，恐怕需要一段时间才能适应这种新风格。而且只有当你能给小型函数很好地命名时，它们才能真正起作用，所以你需要在函数名称下点功夫。人们有时会问我，一个函数多长才算合适？在我看来，长度不是问题，关键在于函数名称和函数本体之间的语义距离（semantic distance ）。如果提炼动作 （extracting ）可以强化代码的清晰度，那就去做，就算函数名称比提炼出来的代码 还长也无所谓。

**做法（Mechanics）**

* 创造一个新函数，根据这个函数的意图来给它命名（以它「做什么」来命名， 而不是以它「怎样做」命名）。
  * 即使你想要提炼（extract）的代码非常简单，例如只是一条消息或一个函数调用，只要新函数的名称能够以更好方式昭示代码意图，你也应该提炼它。但如果你想不出一个更有意义的名称，就别动。
* 将提炼出的代^码从源函数（source）拷贝到新建的目标函数（target）中。
* 仔细检查提炼出的代码，看看其中是否引用了「作用域（scope）限于源函数」的变量（包括局部变量和源函数参数）。
* 检查是否有「仅用于被提炼码」的临时变量（temporary variables ）。如果有，在目标函数中将它们声明为临时变量。
* 检查被提炼码，看看是否有任何局部变量（local-scope variables ）的值被它改变。如果一个临时变量值被修改了，看看是否可以将被提炼码处理为一个查询（query），并将结果赋值给相关变量。如果很难这样做，或如果被修改的 变量不止一个，你就不能仅仅将这段代码原封不动地离炼出来。你可能需要先使用
  [剖解临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_6)
  ，然后再尝试提炼。也可以使用
  [以查询取代临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_8)
  将临时变量消灭掉（请看「范例」中的讨论）。
* 将被提炼码中需要读取的局部变量，当作参数传给目标函数。
* 处理完所有局部变量之后，进行编译。
* 在源函数中，将被提炼码替换为「对目标函数的调用」。
* 如果你将任何临时变量移到目标函数中，请检查它们原本的声明式是否在被提炼码的外围。如果是，现在你可以删除这些声明式了。
* 编译，测试。

---

## 内联函数

一个函数，其本体（method body）应该与其名称（method name\)同样清楚易懂。**在函数调用点插入函数本体，然后移除该函数。**

```
int getRating() {
  return (moreThanFiveLateDeliveries()) ? 2 : 1;
}
boolean moreThanFiveLateDeliveries() {
  return _numberOfLateDeliveries > 5;
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
int getRating() {
  return (_numberOfLateDeliveries > 5) ? 2 : 1;
}
```

**动机**

临时变量的问题在于：它们是暂时的，而且只能在所属函数内使用。由于临时变量只有在所属函数内才可见，所以它们会驱使你写出更长的函数，因为只有这样你才能访问到想要访问的临时变量。如果把临时变量替换为一个查询式（query method），那么同一个class中的所有函数都将可以获得这份信息。这将带给你极大帮助，使你能够为这个编写更清晰的代码。

[以查询取代临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_8)往往是你运用[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)之前必不可少的一个步骤。局部变量会使代码难以被提炼，所以你应该尽可能把它们替换为查询式。

这个重构手法较为直率的情况就是：临时变量只被赋值一次，或者赋值给临时变量的表达式不受其他条件影响。其他情况比较棘手，但也有可能发生。你可能需要先运用[剖解临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_6)或[将查询函数和修改函数分离](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_15)使情况变得简单一些，然后再替换临时变量。如果你想替换的临时变量是用来收集结果的（例如循环中的累加值），你就需要将某些程序逻辑（例如循环）拷贝到查询式（query method）去。

**做法（Mechanics）**

检查函数，确定它不具多态性（is not polymorphic）。

* 如果subclass继承了这个函数,就不要将此函数inline化，因为subclass无法覆写（overridde）一个根本不存在的函数。
* 找出这个函数的所有被调用点。
* 将这个函数的所有被调用点都替换为函数本体（代码）。
* 编译，测试。
* 删除该函数的定义。

---

## 内联临时变量

你有一个临时变量，只被一个简单表达式赋值一次，而它妨碍了其他重构手法。

**将所有对该变量的引用动作，替换为对它赋值的那个表达式本身。**

```
double basePrice = anOrder.basePrice();
return (basePrice > 1000)
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
return (anOrder.basePrice() > 1000)
```

**动机**

[将临时变量内联化](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_3)多半是作为[以查询取代临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_8)的一部分来使用，所以真正的动机出现在后者那儿。惟一单独使用[将临时变量内联化](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_3)的情况是：你发现某个临时变量被赋予某个函数调用的返回值。一般来说，这样的临时变量不会有任何危害，你可以放心地把它在那儿。但如果这个临时变量妨碍了其他的重构 手法——例如[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)，你就应该将它inline化。

**作法（Mechanics）**

* 如果这个临时变量并未被声明为final，那就将它声明为final，然后编译。
* 这可以检查该临时变量是否真的只被赋值一次。
* 找到该临时变量的所有引用点，将它们替换为「为临时变量赋值」之语句中的等号右侧表达式。
* 每次修改后，编译并测试。
* 修改完所有引用点之后，删除该临时变量的声明式和赋值语句。
* 编译，测试。

---

## 以查询取代临时变量

你的程序以一个临时变量（temp）保存某一表达式的运算结果。

**将这个表达式提炼到一个独立函数（译注：所谓查询式，query）中。将这个临时变量的所有「被引用点」替换为「对新函数的调用」。新函数可被其他函数使用。**

```
double basePrice = _quantity * _itemPrice;
if (basePrice > 1000)
    return basePrice * 0.95;
else
    return basePrice * 0.98;
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
if (basePrice() > 1000)
    return basePrice() * 0.95;
else
    return basePrice() * 0.98;

...

double basePrice() {
    return _quantity * _itemPrice;
}
```

**动机（Motivation）**

临时变量的问题在于：它们是暂时的，而且只能在所属函数内使用。由于临时变量只有在所属函数内才可见，所以它们会驱使你写出更长的函数，因为只有这样你才能访问到想要访问的临时变量。如果把临时变量替换为一个查询式（query method），那么同一个class中的所有函数都将可以获得这份信息。这将带给你极大帮助，使你能够为这个编写更清晰的代码。

[以查询取代临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_8)往往是你运用[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)之前必不可少的一个步骤。局部变量会使代码难以被提炼，所以你应该尽可能把它们替换为查询式。

这个重构手法较为直率的情况就是：临时变量只被赋值一次，或者赋值给临时变量的表达式不受其他条件影响。其他情况比较棘手，但也有可能发生。你可能需要先运用[剖解临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_6)或[将查询函数和修改函数分离](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_15)使情况变得简单一些，然后再替换临时变量。如果你想替换的临时变量是用来收集结果的（例如循环中的累加值），你就需要将某些程序逻辑（例如循环）拷贝到查询式（query method）去。

**做法（Mechanics）**

首先是简单情况：

* 找出只被赋值一次的临时变量。
* 如果某个临时变量被赋值超过一次，考虑使用
  [剖解临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_6)
  将它分割成多个变量。
* 将该临时变量声明为final。
* 编译。
* 这可确保该临时变量的确只被赋值一次。
* 将「对该临时变量赋值」之语句的等号右侧部分提炼到一个独立函数中。
* 首先将函数声明为private。日后你可能会发现有更多class需要使用 它，彼时你可轻易放松对它的保护。
* 确保提炼出来的函数无任何连带影响（副作用），也就是说该函数并不修改任何对象内容。如果它有连带影响，就对它进行
  [将查询函数和修改函数分离](http://wangvsa.github.io/refactoring-cheat-sheet/making-method-calls-simpler/#_15)
  。
* 编译，测试。
* 在该临时变量身上实施
  [以查询取代临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_8)。

我们常常使用临时变量保存循环中的累加信息。在这种情况下，整个循环都可以被提为一个独立函数，这也使原本的函数可以少掉几行扰人的循环码。有时候，你可能会用单一循环累加好几个值，就像本书p.26的例子那样。这种情况下你应该针对每个累加值重复一遍循环，这样就可以将所有临时变量都替换为查询式（query）。当然，循环应该很简单，复制这些代码时才不会带来危险。

运用此手法，你可能会担心性能问题。和其他性能问题一样，我们现在不管它，因 为它十有八九根本不会造成任何影响。如果性能真的出了问题，你也可以在优化时期解决它。如果代码组织良好，那么你往往能够发现更有效的优化方案；如果你没有进行重构，好的优化方案就可能与你失之交臂。如果性能实在太糟糕，要把临时变量放回去也是很容易的。

---

## 引入解释性变量

你有一个复杂的表达式。

**将该表达式（或其中一部分）的结果放进一个临时变量，以此变量名称来解释表达式用途。**

```
if ( (platform.toUpperCase().indexOf("MAC") > -1) && (browser.toUpperCase().indexOf("IE") > -1) &&
    wasInitialized() && resize > 0 ) {
    // do something
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
final boolean isMacOs     = platform.toUpperCase().indexOf("MAC") > -1;
final boolean isIEBrowser = browser.toUpperCase().indexOf("IE")  > -1;
final boolean wasResized  = resize > 0;

if (isMacOs && isIEBrowser && wasInitialized() && wasResized) {
    // do something
}
```

**动机（Motivation）**

表达式有可能非常复杂而难以阅读。这种情况下，临时变量可以帮助你将表达式分解为比较容易管理的形式。

在条件逻辑（conditional logic ）中，[引入解释性变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_5)特别有价值：你可以用这项重构将每个条件子句提炼出来，以一个良好命名的临时变量来解释对应条件子句的意义。使用这项重构的另一种情况是，在较长算法中，可以运用临时变量来解释每一步运算的意义。

[引入解释性变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_5)是一个很常见的重构手法，但我得承认，我并不常用它。我几乎总是尽量使用[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)来解释一段代码的意义。毕竟临时变量只在它所处的那个函数中才有意义，局限性较大，函数则可以在对象的整个生命中都有用，并且可被其他对象使用。但有时候，当局部变量使[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)难以进行时，我就使用[引入解释性变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_5)。

**做法（Mechanics）**

* 声明一个final临时变量，将待分解之复杂表达式中的一部分动作的运算结果赋值给它。
* 将表达式中的「运算结果」这一部分，替换为上述临时变量。
  * 如果被替换的这一部分在代码中重复出现，你可以每次一个，逐一替换。
* 编译，测试。
* 重复上述过程，处理表达式的其他部分。

---

## 分解临时变量

你的程序有某个临时变量被赋值超过一次，它既不是循环变量，也不是一个集用临时变量（collecting temporary variable）。

**针对每次赋值，创造一个独立的、对应的临时变量。**

```
double temp = 2 * (_height + _width);
System.out.println (temp);
temp = _height * _width;
System.out.println (temp);
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
final double perimeter = 2 * (_height + _width);
System.out.println (perimeter);
final double area = _height * _width;
System.out.println (area);
```

## 移除对参数的赋值

## 以函数对象取代函数

## 替换算法

## 参考:

[http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/)  
[https://github.com/wangvsa/refactoring-cheat-sheet](https://github.com/wangvsa/refactoring-cheat-sheet)


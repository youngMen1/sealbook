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

**动机（Motivation）**

临时变量有各种不同用途，其中某些用途会很自然地导致临时变量被多次赋值。「循环变量」和「集用临时变量」就是两个典型例子：循环变量（loop variables ）\[Beck\]会随循环的每次运行而改变〔例如for \(int i=0; i&lt;10; i++\)语句中的i〕；集用临时变量（collecting temporary variable）\[Beck\]负责将「通过整个函数的运算」而构成的某个值收集起来。

除了这两种情况，还有很多临时变量用于保存一段冗长代码的运算结果，以便稍后使用。这种临时变量应该只被赋值一次。如果它们被赋值超过一次，就意味它们在函数中承担了一个以上的责任。如果临时变量承担多个责任，它就应该被替换（剖 解）为多个临时变量，每个变量只承担一个责任。同一个临时变量承担两件不同的 事情，会令代码阅读者糊涂。

**做法（Mechanics）**

* 在「待剖解」之临时变量的声明式及其第一次被赋值处，修改其名称。

* 如果稍后之赋值语句是「i = i +某表达式」形式，就意味这是个集用临时变量，那么就不要剖解它。集用临时变量的作用通常是累加、字符串接合、写入stream或者向群集（collection）添加元素。

* 将新的临时变量声明为final。

* 以该临时变量之第二次赋值动作为界，修改此前对该临时变量的所有引用点，让它们引用新的临时变量。

* 在第二次赋值处，重新声明原先那个临时变量。

* 编译，测试。

* 逐次重复上述过程。每次都在声明处对临时变量易名，并修改下次赋值之前的引用点。

---

## 移除对参数的赋值

你的代码对一个参数进行赋值动作。

**以一个临时变量取代该参数的位置。**

```
int discount (int inputVal, int quantity, int yearToDate) {
    if (inputVal > 50) inputVal -= 2;
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
int discount (int inputVal, int quantity, int yearToDate) {
    int result = inputVal;
    if (inputVal > 50) result -= 2;
```

**动机（Motivation）**

首先，我要确定大家都清楚「对参数赋值」这个说法的意思。如果你把一个名为foo 的对象作为参数传给某个函数，那么「对参数赋值」意味改变foo，使它引用（参考、指涉、指向）另一个对象。如果你在「被传入对象」身上进行什么操作，那没问题，我也总是这样干。我只针对「foo被改而指向（引用）完全不同的另一个对象」这种情况来讨论：

```
void aMethod(Object foo) {
    foo.modifyInSomeWay();           // that's OK
    foo = anotherObject;             // trouble and despair will follow you
```

我之所以不喜欢这样的作法，因为它降低了代码的清晰度，而且混淆了 pass by value（传值〕和 pass by reference \(传址）这两种参数传递方式。Java只采用 pass by value传递方式（稍后讨论），我们的讨论也正是基于这一点。

在 pass by value情况下，对参数的任何修改，都不会对调用端造成任何影响。那些用过 pass by reference的人可能会在这一点上犯糊涂。

另一个让人糊涂的地方是函数本体内。如果你只以参数表示「被传递进来的东西」，那么代码会清晰得多，因为这种用法在所有语言中都表现出相同语义。

在Java中，不要对参数赋值；如果你看到手上的代码已经这样做了，请使用[移除对参数的赋值](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_7)。

当然，面对那些使用「输出式参数」（ output parameters）的语言，你不必遵循这条规则。不过在那些语言中我会尽量少用输出式参数。

**做法（Mechanics）**

* 建立一个临时变量，把待处理的参数值赋予它。

* 以「对参数的赋值动作」为界，将其后所有对此参数的引用点，全部替换为「对此临时变量的引用动作」。

* 修改赋值语句，使其改为对新建之临时变量赋值。

* 编译，测试。

* 如果代码的语义是 pass by reference，请在调用端检查调用后是否还使用了这个参数。也要检查有多少个 pass by reference参数「被赋值后又被使用」。请尽量只以return方式返回一个值。如果需要返回的值不只一个，看看可否把需返回的大堆数据变成单一对象，或千脆为每个返回值设计对应的一个独立函数。

---

## 以函数对象取代函数

你有一个大型函数，其中对局部变量的使用，使你无法釆用[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)。

**将这个函数放进一个单独对象中，如此一来局部变量就成了对象内的值域（field） 然后你可以在同一个对象中将这个大型函数分解为数个小型函数。**

```
class Order...
double price() {
    double primaryBasePrice;
    double secondaryBasePrice;
    double tertiaryBasePrice;
    // long computation;
    ...
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

![](/assets/微信截图_20190725194742.png)

**动机（Motivation）**

我在本书中不断向读者强调小型函数的优美动人。只要将相对独立的代码从大型函数中提炼出来，就可以大大提高代码的可读性。

但是，局部变量的存在会增加函数分解难度。如果一个函数之中局部变量泛滥成灾, 那么想分解这个函数是非常困难的。[以查询取代临时变量](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_8)可以助你减轻这一负担，但有时候你会发现根本无法拆解一个需要拆解的函数。这种情况下，你应该把手深深地伸入你的工具箱（好酒沉瓮底呢），祭出函数对象（method object ）\[Beck\]这件法宝。

[以函数对象取代函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_9)会将所有局部变量都变成函数对象（method object）的值域（field）。然后你就可以对这个新对象使用[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)创造出新函数，从而将原本的大型函数拆解变短。

**做法（Mechanics）**

我厚着脸皮从Kent Beck \[Beck\]那里偷来了下列作法：

* 建立一个新class，根据「待被处理之函数」的用途，为这个class命名。

* 在新class中建立一个final值域，用以保存原先大型函数所驻对象。我们将这个值域称为「源对象」。同时，针对原（旧）函数的每个临时变量和每个参数，在新中建立一个个对应的值域保存之。

* 在新class中建立一个构造函数（constructor），接收源对象及原函数的所有参数作为参数。

* 在新class中建立一个compute\(\)函数。

* 将原（旧）函数的代码拷贝到compute\(\)函数中。如果需要调用源对象的任何函数，请以「源对象」值域调用。

* 编译。

* 将旧函数的函数本体替换为这样一条语句：「创建上述新的一个新对象， 而后调用其中的compute\(\)函数」。

现在进行到很有趣的部分了。由于所有局部变量现在都成了值域，所以你可以任意分解这个大型函数，不必传递任何参数。

---

## 替换算法

把某个算法替换为另一个更清晰的算法。

**将函数本体（method body）替换为另一个算法。**

```
String foundPerson(String[] people){
    for (int i = 0; i < people.length; i++) {
        if (people[i].equals ("Don")){
           return "Don";
        }
        if (people[i].equals ("John")){
           return "John";
        }
        if (people[i].equals ("Kent")){
           return "Kent";
        }
    }
    return "";
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
String foundPerson(String[] people){
    List candidates = Arrays.asList(new String[] {"Don", "John", "Kent"});
    for (int i=0; i<people.length; i++)
    if (candidates.contains(people[i]))
        return people[i];
    return "";
}
```

**动机（Motivation）**

我没试过给猫剥皮，不过我听说这有好几种方法，我敢打赌其中某些方法会比另一 些简单。算法也是如此。如果你发现做一件事可以有更清晰的方式，就应该以较清晰的方式取代复杂方式。「重构」可以把一些复杂东西分解为较简单的小块，但有 时你就是必须壮士断腕，删掉整个算法，代之以较简单的算法。随着对问题有了更 多理解，你往往会发现，在你的原先作法之外，有更简单的解决方案，此时你就需 要改变原先的算法。如果你开始使用程序库，而其中提供的某些功能/特性与你自 己的代码重复，那么你也需要改变原先的算法。

有时候你会想要修改原先的算法，让它去做一件与原先动作略有差异的事。这时候你也可以先把原先的算法替换为一个较易修改的算法，这样后续的修改会轻松许多。

使用这项重构手法之前，请先确定自己已经尽可能分解了原先函数。替换一个巨大而复杂的算法是非常困难的，只有先将它分解为较简单的小型函数，然后你才能很有把握地进行算法替换工作。

## 参考:

[http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/)  
[https://github.com/wangvsa/refactoring-cheat-sheet](https://github.com/wangvsa/refactoring-cheat-sheet)


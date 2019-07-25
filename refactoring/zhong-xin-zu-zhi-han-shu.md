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

## 

## 以查询取代临时变量

---

## 引入解释性变量

---

## 分解临时变量

## 移除对参数的赋值

## 以函数对象取代函数

## 替换算法

## 参考:

### [http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/)

### [https://github.com/wangvsa/refactoring-cheat-sheet](https://github.com/wangvsa/refactoring-cheat-sheet)




## 分解条件式 {#_3}

在条件式的每个分支上有着相同的一段代码。

**将这段重复代码搬移到条件式之外。**

```
if (isSpecialDeal()) {
    total = price * 0.95;
    send();
}
else {
    total = price * 0.98;
    send();
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
if (isSpecialDeal())
    total = price * 0.95;
else
    total = price * 0.98;
send();
```

**动机（Motivation）**

有时你会发现，一组条件式的所有分支都执行了相同的某段代码。如果是这样，你就应该将这段代码搬移到条件式外面。这样，代码才能更清楚地表明哪些东西随条件的变化而变化、哪些东西保持不变。

**做法（Mechanics）**

* 鉴别出「执行方式不随条件变化而变化」的代码。
* 如果这些共通代码位于条件式起始处，就将它移到条件式之前。
* 如果这些共通代码位于条件式尾端，就将它移到条件式之后。
* 如果这些共通代码位于条件式中段，就需要观察共通代码之前或之后的代码 是否改变了什么东西。如果的确有所改变，应该首先将共通代码向前或向后 移动，移至条件式的起始处或尾端，再以前面所说的办法来处理。
* 如果共通代码不止一条语句，你应该首先使用以
  [提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)
  将共通 代码提炼到一个独立函数中，再以前面所说的办法来处理。

---

## 合并条件表达式

你有一系列条件测试，都得到相同结果。

**将这些测试合并为一个条件式，并将这个条件式提炼成为一个独立函数。**

```
double disabilityAmount() {
    if (_seniority < 2) return 0;
    if (_monthsDisabled > 12) return 0;
    if (_isPartTime) return 0;
    // compute the disability amount
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
double disabilityAmount() {
    if (isNotEligableForDisability()) return 0;
    // compute the disability amount
```

**动机（Motivation）**

有时你会发现这样一串条件检查：检查条件各不相同，最终行为却一致。如果发现这种情况，就应该使用logical-AND 和logical-OR 将它们合并为一个条件式。

之所以要合并条件代码，有两个重要原因。首先，合并后的条件代码会告诉你「实际上只有一次条件检查，只不过有数个并列条件需要检查而已」，从而使这一次检查的用意更清晰。当然，合并前和合并后的代码有着相同的效果，但原先代码传达出的信息却是「这里有一些独立条件测试，它们只是恰好同时发生」。其次，这项重构往往可以为你使用[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)做好准备。「将检查条件提炼成一个独立函数」对于厘清代码意义非常有用，因为它把描述「做什么」的语句换成了「为什么这样做」。

条件语句的「合并理由」也同时指出了「不要合并」的理由。如果你认为这些检查的确彼此独立，的确不应该被视为同一次检查，那么就不要使用本项重构。因为在 这种情况下，你的代码己经清楚表达出自己的意义。

**作法（Mechanics）**

* 确定这些条件语句都没有副作用（连带影响）。
  * 如果条件式有副作用，你就不能使用本项重构。
* 使用适当的逻辑操作符（logical operators），将一系列相关条件式合并为一个。
* 编译，测试。
* 对合并后的条件式实施
  [提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)
  。

**范例：Ors**

请看下列代码：

```
double disabilityAmount() {
    if (_seniority < 2) return 0;
    if (_monthsDisabled > 12) return 0;
    if (_isPartTime) return 0;
    // compute the disability amount
```

在这段代码中，我们看到一连串的条件检查，它们都做同一件事。对于这样的代码， 上述条件检查等价于一个以"logical-OR"连接起来的语句：

```
double disabilityAmount() {
    if ((_seniority < 2) || (_monthsDisabled > 12) || (_isPartTime)) return 0;
    // compute the disability amount
    ...
```

现在，我可以观察这个新的条件式，并运用

[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)

将它提炼成一个独立函数，以函数名称表达该语句所检查的条件：

```
double disabilityAmount() {
    if (isNotEligibleForDisability()) return 0;
    // compute the disability amount
    ...
}

boolean isNotEligibleForDisability() {
    return ((_seniority < 2) || (_monthsDisabled > 12) || (_isPartTime));
}
```

---

## 合并重复的条件片段 {#_2}

在条件式的每个分支上有着相同的一段代码。

**将这段重复代码搬移到条件式之外。**

```
if (isSpecialDeal()) {
    total = price * 0.95;
    send();
}
else {
    total = price * 0.98;
    send();
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
if (isSpecialDeal())
    total = price * 0.95;
else
    total = price * 0.98;
send();
```

**动机（Motivation）**

有时你会发现，一组条件式的所有分支都执行了相同的某段代码。如果是这样，你就应该将这段代码搬移到条件式外面。这样，代码才能更清楚地表明哪些东西随条件的变化而变化、哪些东西保持不变。

**作法（Mechanics）**

* 鉴别出「执行方式不随条件变化而变化」的代码。
* 如果这些共通代码位于条件式起始处，就将它移到条件式之前。
* 如果这些共通代码位于条件式尾端，就将它移到条件式之后。
* 如果这些共通代码位于条件式中段，就需要观察共通代码之前或之后的代码 是否改变了什么东西。如果的确有所改变，应该首先将共通代码向前或向后 移动，移至条件式的起始处或尾端，再以前面所说的办法来处理。
* 如果共通代码不止一条语句，你应该首先使用以
  [提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)
  将共通 代码提炼到一个独立函数中，再以前面所说的办法来处理。

**范例：（Example）**

你可能遇到这样的代码：

```
if (isSpecialDeal()) {
    total = price * 0.95;
    send();
}
else {
    total = price * 0.98;
    send();
}
```

由于条件式的两个分支都执行了 send\(\) 函数，所以我应该将send\(\) 移到条件式的外围：

```
if (isSpecialDeal())
    total = price * 0.95;
else
    total = price * 0.98;
send();
```

我们也可以使用同样的手法来对待异常（exceptions）。如果在try 区段内「可能引发异常」的语句之后，以及所有catch 区段之内，都重复执行了同一段代码，我就 可以将这段重复代码移到final 区段。

---

## 移出控制标记 {#_5}

在一系列布尔表达式（boolean expressions）中，某个变量带有「控制标记」（control flag）的作用。**以break 语句或return 的语句取代控制标记。**

**动机（Motivation）**

在一系列条件表达式中，你常常会看到「用以判断何时停止条件检查」的控制标记（control flag）：

```
set done to false
while not done
    if (condition)
        do something
    set done to true
    next step of loop
```

这样的控制标记带来的麻烦超过了它所带来的便利。人们之所以会使用这样的控制标记，因为结构化编程原则告诉他们：每个子程序（routines）只能有一个入口（entry） 和一个出口（exit）。我赞同「单一入口」原则（而且现代编程语言也强迫我们这样做），但是「单一出口」原则会让你在代码中加入讨厌的控制标记，大大降低条件表达式的可读性。这就是编程语言提供break 语句和continue 语句的原因：你可以用它们跳出复杂的条件语句。去掉控制标记所产生的效果往往让你大吃一惊：条件语句真正的用途会清晰得多。

**作法（Mechanics）**

对控制标记（control flags）的处理，最显而易见的办法就是使用Java 提供的break 语句或continue 语句。

* 找出「让你得以跳出这段逻辑」的控制标记值。
* 找出「将可跳出条件式之值赋予标记变量」的那个语句，代以恰当的break 语句或continue 语句。
* 每次替换后，编译并测试。

在未能提供break 和continue 语句的编程语言中，我们可以使用另一种办法：

* 运用
  [提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)
  ，将整段逻辑提炼到一个独立函数中。
* 找出「让你得以跳出这段逻辑」的那些控制标记值。
* 找出「将可跳出条件式之值赋予标记变量」的那个语句，代以恰当的return 语句。
* 每次替换后，编译并测试。

即使在支持break 和continue 语句的编程语言中，我通常也优先考虑上述第二方案。因为return 语句可以非常清楚地表示：不再执行该函数中的其他任何代码。 如果还有这一类代码，你早晚需要将这段代码提炼出来。

请注意标记变量是否会影响这段逻辑的最后结果。如果有影响，使用break 语句之后你还得保留控制标记值。如果你已经将这段逻辑提炼成一个独立函数，也可以将控制标记值放在return 语句中返回。

**范例：以break 取代简单的控制标记**

下列函数用来检查一系列人名之中是否包含两个可疑人物的名字（这两个人的名字硬编码于代码中〕：

```
void checkSecurity(String[] people) {
    boolean found = false;
    for (int i = 0; i < people.length; i++) {
        if (! found) {
            if (people[i].equals ("Don")){
                sendAlert();
                found = true;
            }
            if (people[i].equals ("John")){
                sendAlert();
                found = true;
            }
        }
    }
}
```

这种情况下很容易找出控制标记：当变量found 被赋予true 时，搜索就结束。我可以逐一引入break 语句：

```
void checkSecurity(String[] people) {
    boolean found = false;
    for (int i = 0; i < people.length; i++) {
        if (! found) {
            if (people[i].equals ("Don")){
                sendAlert();
                break;
            }
            if (people[i].equals ("John")){
                sendAlert();
                found = true;
            }
        }
    }
}
```

最后获得这样的成功：

```
void checkSecurity(String[] people) {
    boolean found = false;
    for (int i = 0; i < people.length; i++) {
        if (! found) {
            if (people[i].equals ("Don")){
                sendAlert();
                break;
            }
            if (people[i].equals ("John")){
                sendAlert();
                break;
            }
        }
    }
}
```

然后我就可以把对控制标记的所有引用去掉：

```
void checkSecurity(String[] people) {
    for (int i = 0; i < people.length; i++) {
        if (people[i].equals ("Don")){
            sendAlert();
            break;
        }
        if (people[i].equals ("John")){
            sendAlert();
            break;
        }
    }
}
```

---

## 以卫语句取代嵌套条件式 {#_7}

函数中的条件逻辑（conditional logic）使人难以看清正常的执行路径。

**使用卫语句（guard clauses）表现所有特殊情况。**

```
double getPayAmount() {
    double result;
    if (_isDead) result = deadAmount();
    else {
        if (_isSeparated) result = separatedAmount();
        else {
            if (_isRetired) result = retiredAmount();
            else result = normalPayAmount();
        }
    }
    return result;
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
double getPayAmount() {
    if (_isDead) return deadAmount();
    if (_isSeparated) return separatedAmount();
    if (_isRetired) return retiredAmount();
    return normalPayAmount();
}
```

**动机（Motivation）**

根据我的经验，条件式通常有两种呈现形式。第一种形式是：所有分支都属于正常行为。第二种形式则是：条件式提供的答案中只有一种是正常行为，其他都是不常见的情况。

这两类条件式有不同的用途，这一点应该通过代码表现出来。如果两条分支都是正常行为，就应该使用形如「if…then…」的条件式；如果某个条件极其罕见，就应该单独检查该条件，并在该条件为真时立刻从函数中返回。这样的单独检查常常被称为「卫语句（guard clauses）」\[Beck\]。

[以卫语句取代嵌套条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_7)的精髓就是：给某一条分支以特别的重视。如果使用if-then-else 结构，你对if 分支和else 分支的重视是同等的。 这样的代码结构传递给阅读者的消息就是：各个分支有同样的重要性。卫语句（guard clauses）就不同了，它告诉阅读者：『这种情况很罕见，如果它真的发生了，请做 一些必要的整理工作，然后退出。』

「每个函数只能有一个入口和一个出口」的观念，根深蒂固于某些程序员的脑海里。 我发现，当我处理他们编写的代码时，我经常需要使用[以卫语句取代嵌套条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_7)。现今的编程语言都会强制保证每个函数只有一个入口， 至于「单一出口」规则，其实不是那么有用。在我看来，保持代码清晰才是最关键的：如果「单一出口」能使这个函数更清楚易读，那么就使用单一出口；否则就不必这么做。

**作法（Mechanics）**

* 对于每个检查，放进一个卫语句（guard clauses）。
  * 卫语句要不就从函数中返回，要不就抛出一个异常（exception）。
* 每次将「条件检查」替换成「卫语句」后，编译并测试。
  * 如果所有卫语句都导致相同结果，请使用
    [合并条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_1)
    s。

**范例：（Example）**

想像一个薪资系统，其中以特殊规则处理死亡员工、驻外员工、退休员工的薪资。这些情况不常有，但的确偶而会出现。

假设我在这个系统中看到下列代码：

```
double getPayAmount() {
    double result;
    if (_isDead) result = deadAmount();
    else {
        if (_isSeparated) result = separatedAmount();
        else {
            if (_isRetired) result = retiredAmount();
            else result = normalPayAmount();
        }
    }
    return result;
}
```

在这段代码中，非正常情况的检查掩盖了正常情况的检查，所以我应该使用『卫语句」来取代这些检查，以提高程序清晰度。我可以逐一引入卫语句。让我们从最上面的条件检查动作开始：

```
double getPayAmount() {
    double result;
    if (_isDead) return deadAmount();
    if (_isSeparated) result = separatedAmount();
    else {
        if (_isRetired) result = retiredAmount();
        else result = normalPayAmount();
    }
    return result;
 }
```

然后，继续下去，仍然一次替换一个检查动作：

```
double getPayAmount() {
    double result;
    if (_isDead) return deadAmount();
    if (_isSeparated) return separatedAmount();
    if (_isRetired) result = retiredAmount();
    else result = normalPayAmount();
    return result;
}
```

然后是最后一个：

```
double getPayAmount() {
    double result;
    if (_isDead) return deadAmount();
    if (_isSeparated) return separatedAmount();
    if (_isRetired) return retiredAmount();
    result = normalPayAmount();
    return result;
}
```

此时，result 变量已经没有价值了，所以我把它删掉：

```
double getPayAmount() {
    if (_isDead) return deadAmount();
    if (_isSeparated) return separatedAmount();
    if (_isRetired) return retiredAmount();
    return normalPayAmount();
}
```

嵌套（nested）条件代码往往由那些深信「每个函数只能有一个出口」的程序员写出。我发现那条规则（函数只能有一个出口）实在有点太简单化了。如果对函数剩余部分不再有兴趣，当然应该立刻退出。引导阅读者去看一个没有用的else 区段，只会妨碍他们的理解。

**范例：将条件逆反（Reversing the Conditions）**

审阅本书初稿时，Joshua Kerievsky 指出：你常常可以将条件表达式逆反，从而实现[以卫语句取代嵌套条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_7)。为了拯救我可怜的想像力，他还好心帮我想了个例子：

```
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital > 0.0) {
        if (_intRate > 0.0 && _duration > 0.0) {
            result = (_income / _duration) * ADJ_FACTOR;
        }
    }
    return result;
}
```

同样地，我逐一进行替换。不过这次在插入卫语句（guard clauses）时，我需要将相应的条件逆反过来：

```
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital <= 0.0) return result;
    if (_intRate > 0.0 && _duration > 0.0) {
        result = (_income / _duration) * ADJ_FACTOR;
    }
    return result;
}
```

下一个条件稍微复杂一点，所以我分两步进行逆反。首先加入一个"logical-NOT"操作：

```
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital <= 0.0) return result;
    if (!(_intRate > 0.0 && _duration > 0.0)) return result;
    result = (_income / _duration) * ADJ_FACTOR;
    return result;
}
```

但是在这样的条件式中留下一个"logical-NOT"，会把我的脑袋拧成一团乱麻，所以我把它简化成下面这样：

```
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital <= 0.0) return result;
    if (_intRate <= 0.0 || _duration <= 0.0) return result;
    result = (_income / _duration) * ADJ_FACTOR;
    return result;
}
```

这时候我比较喜欢在卫语句（guard clause）内返回一个明确值，因为这样我可以一 目了然地看到卫语句返回的失败结果。此外，这种时候我也会考虑使用[以符号常量/字面常量取代魔法数](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_10)。

```
public double getAdjustedCapital() {
    double result = 0.0;
    if (_capital <= 0.0) return 0.0;
    if (_intRate <= 0.0 || _duration <= 0.0) return 0.0;
    result = (_income / _duration) * ADJ_FACTOR;
    return result;
}
```

完成替换之后，我同样可以将临时变量移除：

```
public double getAdjustedCapital() {
    if (_capital <= 0.0) return 0.0;
    if (_intRate <= 0.0 || _duration <= 0.0) return 0.0;
    return (_income / _duration) * ADJ_FACTOR;
}
```

---

## 以多态取代条件式 {#_6}

你手上有个条件式，它根据对象型别的不同而选择不同的行为。

**将这个条件式的每个分支放进一个subclass 内的覆写函数中，然后将原始函数声明为抽象函数（abstract method）。**

```
double getSpeed() {
    switch (_type) {
    case EUROPEAN:
        return getBaseSpeed();
    case AFRICAN:
        return getBaseSpeed() - getLoadFactor() * _numberOfCoconuts;
    case NORWEGIAN_BLUE:
        return (_isNailed) ? 0 : getBaseSpeed(_voltage);
    }
    throw new RuntimeException ("Should be unreachable");
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/09fig01a.gif)

**动机（Motivation）**

在面向对象术语中，听上去最高贵的词非「多态」莫属。多态（polymorphism）最根本的好处就是：如果你需要根据对象的不同型别而采取不同的行为，多态使你不必编写明显的条件式（explicit conditional ）。

正因为有了多态，所以你会发现：「针对type code（型别码）而写的switch 语句」 以及「针对type string （型别名称字符串）而写的if-then-else 语句」在面向对象程序中很少出现。

多态（polymorphism）能够给你带来很多好处。如果同一组条件式在程序许多地点出现，那么使用多态的收益是最大的。使用条件式时，如果你想添加一种新型别，就必须查找并更新所有条件式。但如果改用多态，只需建立一个新的subclass ，并在其中提供适当的函数就行了。class 用户不需要了解这个subclass ，这就大大降低了系统各部分之间的相依程度，使系统升级更加容易。

**作法（Mechanics）**

使用[以多态取代条件式](http://wangvsa.github.io/refactoring-cheat-sheet/simplifying-conditional-expressions/#_6)之前，你首先必须有一个继承结构。你可能已经通过先前的重构得到了这一结构。如果还没有，现在就需要建立它。

要建立继承结构，你有两种选择：[以子类取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#_14)和[以State/Strategy取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#statestrategy)。前一种作法比较简单，因此你应该尽可能使用它。但如果你需要在对象创建好之后修改type code；就不能使用subclassing 作法，只能使用State/Strategy 模式。此，如果由于其他原因你要重构的class 已经有了subclass ，那么也得使用State/Strategy 。记住，如果若干switch 语句针对的是同一个type code；你只需针对这个type code 建立一个继承结构就行 了。

现在，可以向条件式开战了。你的目标可能是switch（case）语句，也可能是if 语句。

* 如果要处理的条件式是一个更大函数中的一部分，首先对条件式进行分析，然后使用
  [提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)
  将它提炼到一个独立函数去。
* 如果有必要，使用
  [搬移函数](http://wangvsa.github.io/refactoring-cheat-sheet/moving-features-between-objects/#_3)
  将条件式放置到继承结构的顶端。
* 任选一个subclass ，在其中建立一个函数，使之覆写superclass 中容纳条件式的那个函数。将「与subclass 相关的条件式分支」拷贝到新建函数中，并对它进行适当调整。
  * 为了顺利进行这一步骤，你可能需要将superclass 中的某些private 值域声明为protected 。
* 编译，测试。
* 在superclass 中删掉条件式内被拷贝出去的分支。
* 编译，测试。
* 针对条件式的每个分支，重复上述过程，直到所有分支都被移到subclass 内的函数为止。
* 将superclass 之中容纳条件式的函数声明为抽象函数（abstract method）。

**范例：（Example）**





##  {#null}

## 引入Null对象 {#null}

## 引入断言 {#_4}




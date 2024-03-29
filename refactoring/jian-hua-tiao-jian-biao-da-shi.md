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

请允许我继续使用「员工与薪资」这个简单而又乏味的例子。我的classes是从[以State/Strategy取代型别码](http://wangvsa.github.io/refactoring-cheat-sheet/organizing-data/#statestrategy)那个例子中拿来的，因此示意图就如图9.1所示（如果想知道这个图是怎么得到的，请看第8章范例）。

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/09fig01.gif)

图9.1 继承机构

```
class Employee...
    int payAmount() {
        switch (getType()) {
        case EmployeeType.ENGINEER:
            return _monthlySalary;
        case EmployeeType.SALESMAN:
            return _monthlySalary + _commission;
        case EmployeeType.MANAGER:
            return _monthlySalary + _bonus;
        default:
            throw new RuntimeException("Incorrect Employee");
        }
    }
    int getType() {
        return _type.getTypeCode();
    }
    private EmployeeType _type;

    abstract class EmployeeType...
    abstract int getTypeCode();

class Engineer extends EmployeeType...
    int getTypeCode() {
        return Employee.ENGINEER;
    }

... and other subclasses
```

switch 语句已经被很好地提炼出来，因此我不必费劲再做一遍。不过我需要将它移至EmployeeType class，因为EmployeeType 才是被subclassing 的class 。

```
class EmployeeType...
    int payAmount(Employee emp) {
        switch (getTypeCode()) {
            case ENGINEER:
                return emp.getMonthlySalary();
            case SALESMAN:
                return emp.getMonthlySalary() + emp.getCommission();
            case MANAGER:
                return emp.getMonthlySalary() + emp.getBonus();
            default:
                throw new RuntimeException("Incorrect Employee");
        }
    }
```

由于我需要EmployeeType class 的数据，所以我需要将Employee 对象作为参数传递给payAmount\(\)。这些数据中的一部分也许可以移到EmployeeType class 来，但那是另一项重构需要关心的问题了。

调整代码，使之通过编译，然后我修改Employee 中的payAmount\(\) 函数，令它委托（delegate，转调用）EmployeeType ：

```
class Employee...
    int payAmount() {
        return _type.payAmount(this);
    }
```

现在，我可以处理switch 语句了。这个过程有点像淘气小男孩折磨一只昆虫——每次掰掉它一条腿 6。首先我把switch 语句中的"Engineer"这一分支拷贝到Engineer class：

```
class Engineer...
    int payAmount(Employee emp) {
        return emp.getMonthlySalary();
    }
```

6译注：「腿」和条件式「分支」的英文都是"leg"。作者幽默地说「掰掉一条腿」， 意思就是「去掉一个分支」。

这个新函数覆写了superclass 中的switch 语句之内那个专门处理"Engineer"的分支。我是个徧执狂，有时我会故意在case 子句中放一个陷阱，检查Engineer class 是否正常工作（是否被调用）：

```
class EmployeeType...
    int payAmount(Employee emp) {
        switch (getTypeCode()) {
        case ENGINEER:
            throw new RuntimeException ("Should be being overridden");
        case SALESMAN:
            return emp.getMonthlySalary() + emp.getCommission();
        case MANAGER:
            return emp.getMonthlySalary() + emp.getBonus();
        default:
            throw new RuntimeException("Incorrect Employee");
    }
}
```

接下来，我重复上述过程，直到所有分支都被去除为止：

```
class Salesman...
    int payAmount(Employee emp) {
        return emp.getMonthlySalary() + emp.getCommission();
    }

class Manager...
    int payAmount(Employee emp) {
        return emp.getMonthlySalary() + emp.getBonus();
    }
```

然后，将superclass 的payAmount\(\) 函数声明为抽象函数：

```
class EmployeeType...
    abstract int payAmount(Employee emp);
```

---

## 引入Null对象 {#null}

你需要再三检查「某物是否为null value」。

**将null value （无效值）替换为null object（无效物）。**

```
if (customer == null) plan = BillingPlan.basic();
else plan = customer.getPlan();
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/09fig01b.gif)

**动机（Motivation）**

多态（polymorphism ）的最根本好处在于：你不必再向对象询问「你是什么型别」 而后根据得到的答案调用对象的某个行为——你只管调用该行为就是了，其他的一切多态机制会为你安排妥当。当你的某个值域内容是null value 时，多态可扮演另一个较不直观（亦较不为人所知）的用途。让我们先听听Ron Jeffries 的故事。

> Ron Jeffries 我们第一次使用Null Object 模式，是因为Rih Garzaniti 发现，系统在对对象发送一个消息之前，总要检査对象是否存在，这样的检査出现很多次。我们可能会向一个对象索求它所相关的Person 对象，然后再问那个对象是否为null 。如果对象的确存在，我们才能调用它的rate\(\) 函数以查询这个人的薪资级别。我们在好些地方都是这样做的， 造成的重复代码让我们很烦心。 所以.我们编写了一个MissingPerson class，让它返回 '0' 薪资等级（我们把null objects 称为missing object（虚构对象）。很快地MissingPerson 就有了很多函数，rate\(\) 自然是其中之一。如今我们的系统有超过80个null object classes。 我们常常在显示信息的时候使用null object。例如我们想要显示一个Person 对象信息，它大约有20个instance 变量。如果这些变量可被设为null，那么打印一个Person 对象的工作将非常复杂。所以我们不让instance 变量被设为null ，而是插入各式各样的null objects ——它们都知道如何正常（正确地）显示自己。这样，我们就可以摆脱大量代码。 我们对null object 的最聪明运用，就是拿它来表示不存在的Gemstone session。我们使用Gemstone 数据库来保存成品（程序代码），但我们更愿息在没有数据库的情况下进行开发，毎过一周左右再把新码放进Gemstone 数据库。然而在代码的某些地方，我们必须登录（log in）一个Gemstone session。当我们没有Gemstone 数据库时，我们就仅仅安插一个miss Gemstone session，其接口和真正的Gemstone session 一模一样，使我们无需判断数据库是否存在，就可以进行开发和测试。 null object 的另一个用途是表现出「虚构的箱仓」（missing bin）。所谓「箱仓\]，这里是指群集（collection），用来保存某些薪资值，并常常谣要对各个薪资值进行加和或遍历。如果某个箱仓不存在，我们就给出一个虚构的箱仓对象，其行为和一个空箱仓（empty bin）一样；这个虚构箱仓知道自己其实不带任何数据，总值为0。通过这种作法，我们就不必为上千位员工每人产生数十来个空箱（empty bins）对象了。 使用null objects 有个非常有趣的性质：好事绝对不会因为null objects 而「被破坏」。由于null objects 对所有外界请求的响应，都像real objects 的响应一样，所以系统行为总是正常的。但这并非总是好事，有吋会造成问题的侦测和查找上的困难，因为从来没有任何东西被破坏。当然，只要认真检查一下，你就会发现null objects 有时出现在不该出现的地方。 请记住：null objects 一定是常量，它们的任何成分都不会发生变化。因此我们可以使用Singleton 模式\[Gang of Four\]来实现它们。例如不管任何时候，只要你索求一个MissingPerson 对象，你得到的一定是MissingPerson 的惟一实体。

关于Null Object 模式，你可以在Woolf \[Woolf\] 中找到更详细的介绍。

**作法（Mechanics）**

* 为source class 建立一个subclass ，使其行为像source class 的null 版本。在source class 和null class 中都加上isNull\(\) 函数，前者的isNull\(\) 应该返回false，后者的isNull\(\) 应该返回true。
* 下面这个办法也可能对你有所帮助：建立一个nullable 接口，将isNull\(\) 函数放在其中，让source class 实现这个接口。
* 另外，你也可以创建一个testing 接口，专门用来检查对象是否为null。
* 编译。
* 找出所有「索求source object 却获得一个null 」的地方。修改这些地方，使它们改而获得一个null object。
* 找出所有「将source object 与null 做比较」的地方。修改这些地方，使它们调用isNull\(\) 函数。
* 你可以每次只处理一个source object 及其客户程序，编译并测试后， 再处理另一个source object 。
* 你可以在「不该再出现null value」的地方放上一些assertions（断言）， 确保null 的确不再出现。这可能对你有所帮助。
* 编译，测试。
* 找出这样的程序点：如果对象不是null ，做A动作，否则做B 动作。
* 对于每一个上述地点，在null class 中覆写A动作，使其行为和B 动作相同。
* 使用上述的被覆写动作（A），然后删除「对象是否等于null」的条件测试。编译并测试。

---

## 引入断言 {#_4}

某一段代码需要对程序状态（state）做出某种假设。

**以assertion（断言）明确表现这种假设。**

```
double getExpenseLimit() {
    // should have either expense limit or a primary project
    return (_expenseLimit != NULL_EXPENSE) ?
        _expenseLimit:_primaryProject.getMemberExpenseLimit();
}
```

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

```
double getExpenseLimit() {
    Assert.isTrue (_expenseLimit != NULL_EXPENSE || _primaryProject != null);
    return (_expenseLimit != NULL_EXPENSE) ?
        _expenseLimit: _primaryProject.getMemberExpenseLimit();
}
```

**动机（Motivation）**

常常会有这样一段代码：只有当某个条件为真时，该段代码才能正常运行。例如「平方报计算」只对正值才能进行（译注：这里没考虑复数与虚数），又例如某个对象 可能假设其值域（fields）至少有一个不等于null。

这样的假设通常并没有在代码中明确表现出来，你必须阅读整个算法才能看出。有时程序员会以注释写出这样的假设。而我要介绍的是一种更好的技术：使用assertion（断言）明确标明这些假设。

assertion 是一个条件式，应该总是为真。如果它失败，表示程序员犯了错误。因此assertion的失败应该导致一个unchecked exception 7（不可控异常〕。Assertions 绝对不能被系统的其他部分使用。实际上程序最后成品往往将assertions 统统删除。因此，标记「某些东西是个assertion」是很重要的。

7译注：所谓unchecked exception 是指「未曾于函数签名式（signature）中列出」的异常。

Assertions 可以作为交流与调试的辅助。在交流（沟通〕的角度上，assertions 可以帮助程序阅读者理解代码所做的假设；在调试的角度上，assertions 可以在距离「臭虫」最近的地方抓住它们。当我编写自我测试代码的时候，我发现，assertions 在调试方面的帮助变得不那么重要了，但我仍然非常看重它们在交流方面的价值。

**作法（Mechanics）**

如果程序员不犯错，assertions 就应该不会对系统运行造成任何影响，所以加入assertions 永远不会影响程序的行为。

* 如果你发现代码「假设某个条件始终（必须）为真\]，就加入一个assertion 明确说明这种情况。
  * 你可以新建一个Assert class，用于处理各种情况下的assertions 。

注意，不要滥用assertions 。请不要使用它来检查你「认为应该为真」的条件，请只使用它来检查「一定必须为真」的条件。滥用assertions 可能会造成难以维护的重复逻辑。在一段逻辑中加入assertions 是有好处的，因为它迫使你重新考虑这段代 码的约束条件。如果「不满足这些约朿条件，程序也可以正常运行」，assertions 就不会带给你任何帮助，只会把代码变得混乱，并且有可能妨碍以后的修改。

你应该常常问自己：如果assertions 所指示的约束条件不能满足，代码是否仍能正常运行？如果可以，就把assertions 拿掉。

另外，还需要注意assertions 中的重复代码。它们和其他任何地方的重复代码一样不好闻。你可以大胆使用[提炼函数](http://wangvsa.github.io/refactoring-cheat-sheet/composing-methods/#_1)去掉那些重复代码。


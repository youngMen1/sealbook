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

##  {#_7}

## 以卫语句取代嵌套条件式 {#_7}

函数中的条件逻辑（conditional logic）使人难以看清正常的执行路径。

**使用卫语句（guard clauses）表现所有特殊情况。**





## 以多态取代条件式 {#_6}

## 引入Null对象 {#null}

## 引入断言 {#_4}




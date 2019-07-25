## 提炼函数

你有一段代码可以被组织在一起并独立出来。**将这段代码放进一个独立函数中，并让函数名称解释该函数的用途。**

```
void printOwing(double amount) {
    printBanner();
    //print details
    System.out.println ("name:" + _name);
    System.out.println ("amount" + amount);
}
```

```java

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

---

## 内联函数

一个函数，其本体（method body）应该与其名称（method name\)同样清楚易懂。**在函数调用点插入函数本体，然后移除该函数。**

![](http://wangvsa.github.io/refactoring-cheat-sheet/images/arrow.gif)

---

## 内联临时变量

---

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




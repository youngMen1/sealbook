## BigDecimal加减乘除计算

// 尽量用字符串的形式初始化  
BigDecimal num1 = new BigDecimal\("0.005"\);  
BigDecimal num2 = new BigDecimal\("1000000"\);

### 加法 add\(\)函数

* BigDecimal result1 = num1.add\(num2\);
  ### 减法subtract\(\)函数
* BigDecimal result2 = num1.subtract\(num2\);
  ### 乘法multiply\(\)函数
* BigDecimal result3 = num1.multiply\(num2\);
  ### 除法divide\(\)函数\(BigDecimal divisor 除数，int scale 精确小数位，int roundingMode 舍入模式\)
* BigDecimal result5 = num2.divide\(num1,20,BigDecimal.ROUND\_HALF\_UP\);
* 1.可以看到舍入模式有很多种BigDecimal.ROUND\_XXXX\_XXX, 具体都是什么意思呢?
  ![img](/static/image/2018091611573630.png)
* 计算1÷3的结果（最后一种ROUND\_UNNECESSARY在结果为无限小数的情况下会报错）
  ![img](/static/image/2018091611592867.png)

## 八种舍入模式解释如下

### 1、ROUND\_UP

舍入远离零的舍入模式。

在丢弃非零部分之前始终增加数字\(始终对非零舍弃部分前面的数字加1\)。

注意，此舍入模式始终不会减少计算值的大小。

### 2、ROUND\_DOWN

接近零的舍入模式。

在丢弃某部分之前始终不增加数字\(从不对舍弃部分前面的数字加1，即截短\)。

注意，此舍入模式始终不会增加计算值的大小。
无条件进1，1.01 ->1.1
### 3、ROUND\_CEILING

接近正无穷大的舍入模式。

如果 BigDecimal 为正，则舍入行为与 ROUND\_UP 相同;

如果为负，则舍入行为与 ROUND\_DOWN 相同。

注意，此舍入模式始终不会减少计算值。

### 4、ROUND\_FLOOR

接近负无穷大的舍入模式。

如果 BigDecimal 为正，则舍入行为与 ROUND\_DOWN 相同;

如果为负，则舍入行为与 ROUND\_UP 相同。

注意，此舍入模式始终不会增加计算值。

### 5、ROUND\_HALF\_UP

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为向上舍入的舍入模式。

如果舍弃部分 &gt;= 0.5，则舍入行为与 ROUND\_UP 相同;否则舍入行为与 ROUND\_DOWN 相同。

注意，这是我们大多数人在小学时就学过的舍入模式\(四舍五入\)。

### 6、ROUND\_HALF\_DOWN

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为上舍入的舍入模式。

如果舍弃部分 &gt; 0.5，则舍入行为与 ROUND\_UP 相同;否则舍入行为与 ROUND\_DOWN 相同\(五舍六入\)。

### 7、ROUND\_HALF\_EVEN

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则向相邻的偶数舍入。

如果舍弃部分左边的数字为奇数，则舍入行为与 ROUND\_HALF\_UP 相同;

如果为偶数，则舍入行为与 ROUND\_HALF\_DOWN 相同。

注意，在重复进行一系列计算时，此舍入模式可以将累加错误减到最小。

此舍入模式也称为“银行家舍入法”，主要在美国使用。四舍六入，五分两种情况。

如果前一位为奇数，则入位，否则舍去。

以下例子为保留小数点1位，那么这种舍入方式下的结果。

1.15&gt;1.2 1.25&gt;1.2

### 8、ROUND\_UNNECESSARY

断言请求的操作具有精确的结果，因此不需要舍入。

## 绝对值abs\(\)函数

* BigDecimal result4 = num3.abs\(\);
  ## 注意
* 1.System.out.println\(\)中的数字默认是double类型的，double类型小数计算不精准。
* 2.使用BigDecimal类构造方法传入double类型时，计算的结果也是不精确的！
  因为不是所有的浮点数都能够被精确的表示成一个double 类型值，有些浮点数值不能够被精确的表示成 double 类型值，因此它会被表示成与它最接近的 double 类型的值。必须改用传入String的构造方法。这一点在BigDecimal类的构造方法注释中有说明。






## BigDecimal加减乘除计算

 // 尽量用字符串的形式初始化
BigDecimal num1 = new BigDecimal("0.005");
BigDecimal num2 = new BigDecimal("1000000");
* 加法 add()函数 
BigDecimal result1 = num1.add(num2);
* 减法subtract()函数
BigDecimal result2 = num1.subtract(num2);
* 乘法multiply()函数
BigDecimal result3 = num1.multiply(num2);
* 除法divide()函数(BigDecimal divisor 除数，int scale 精确小数位，int roundingMode 舍入模式)
BigDecimal result5 = num2.divide(num1,20,BigDecimal.ROUND_HALF_UP);
1.可以看到舍入模式有很多种BigDecimal.ROUND_XXXX_XXX, 具体都是什么意思呢



* 绝对值abs()函数
BigDecimal result4 = num3.abs();
### 注意
1.System.out.println()中的数字默认是double类型的，double类型小数计算不精准。
2.使用BigDecimal类构造方法传入double类型时，计算的结果也是不精确的！
因为不是所有的浮点数都能够被精确的表示成一个double 类型值，有些浮点数值不能够被精确的表示成 double 类型值，因此它会被表示成与它最接近的 double 类型的值。必须改用传入String的构造方法。这一点在BigDecimal类的构造方法注释中有说明。



## BigDecimal加减乘除计算

 // 尽量用字符串的形式初始化
BigDecimal num12 = new BigDecimal("0.005");
BigDecimal num22 = new BigDecimal("1000000");
* 加法 add()函数
     
* 减法subtract()函数
* 乘法multiply()函数    
* 除法divide()函数    
* 绝对值abs()函数
### 注意
1.System.out.println()中的数字默认是double类型的，double类型小数计算不精准。
2.使用BigDecimal类构造方法传入double类型时，计算的结果也是不精确的！
因为不是所有的浮点数都能够被精确的表示成一个double 类型值，有些浮点数值不能够被精确的表示成 double 类型值，因此它会被表示成与它最接近的 double 类型的值。必须改用传入String的构造方法。这一点在BigDecimal类的构造方法注释中有说明。

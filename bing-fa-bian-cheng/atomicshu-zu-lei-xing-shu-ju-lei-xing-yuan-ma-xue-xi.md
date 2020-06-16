# 1.Atomic数组数据类型源码学习

## 1.1.基本介绍

Atomic包下提供能原子更新数组中元素的类有：

1. AtomicIntegerArray：原子更新整型数组中的元素；
2. AtomicLongArray：原子更新长整型数组中的元素；
3. AtomicReferenceArray：原子更新引用类型数组中的元素

这几个类的用法一致，就以AtomicIntegerArray来总结下常用的方法：

1. addAndGet\(int i, int delta\)：以原子更新的方式将数组中索引为i的元素与输入值相加；
2. getAndIncrement\(int i\)：以原子更新的方式将数组中索引为i的元素自增加1；
3. compareAndSet\(int i, int expect, int update\)：将数组中索引为i的位置的元素进行更新

可以看出，AtomicIntegerArray与AtomicInteger的方法基本一致，只不过在AtomicIntegerArray的方法中会多一个指定数组索引位i。下面举一个简单的例子：




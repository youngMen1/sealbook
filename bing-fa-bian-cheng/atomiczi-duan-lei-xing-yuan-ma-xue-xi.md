# 1.

atomic包提高原子更新基本类型的工具类，主要有这些：

1. AtomicBoolean：以原子更新的方式更新boolean；
2. AtomicInteger：以原子更新的方式更新Integer;
3. AtomicLong：以原子更新的方式更新Long；

这几个类的用法基本一致，这里以AtomicInteger为例总结常用的方法

1. addAndGet\(int delta\) ：以原子方式将输入的数值与实例中原本的值相加，并返回最后的结果；
2. incrementAndGet\(\) ：以原子的方式将实例中的原值进行加1操作，并返回最终相加后的结果；
3. getAndSet\(int newValue\)：将实例中的值更新为新值，并返回旧值；
4. getAndIncrement\(\)：以原子的方式将实例中的原值加1，返回的是自增前的旧值；




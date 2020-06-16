# 1.AtomicInteger 源码分析

## 1.1.TOP 带着问题看源码 {#TOP-%E5%B8%A6%E7%9D%80%E9%97%AE%E9%A2%98%E7%9C%8B%E6%BA%90%E7%A0%81}

1. AtomicInteger 是怎么做到线程安全的
2. AtomicInteger 是怎么实现自增的

## 1.2.基本介绍

AtomicInteger 扩展了 Number，适用于基于数字的处理，并提供了如原子递增等，适合一些计数场景

```
// Integer类型（原子性）
public class AtomicInteger extends Number implements Serializable {
.........
}
```

```
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```

可以看到 value 是采用 volatile 修饰的，并通过 Unsafe 类获取 value 的偏移量，方便后续使用 CAS 操作

## 1.3. 自增 & 自减 {#2.-%E8%87%AA%E5%A2%9E-&-%E8%87%AA%E5%87%8F}

```
// 获取 & 自增
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
// 自增 & 获取
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
// 获取 & 自减
public final int getAndDecrement() {
    return unsafe.getAndAddInt(this, valueOffset, -1);
}
// 自减 & 获取
public final int decrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, -1) - 1;
}
```

AtomicInteger 提供了自增/自减的两个场景方法，一个返回旧值，一个返回新增/自减后的。

实际都是通过Unsafe 的 getAndAddInt 方法来实现的，可以看到实际上 getAndAddInt 就是一个 cas + 自旋操作来实现。

```
// 返回对象var1的offset地址处的值，并将该值原子性地增加delta
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
       // 获取对象var1中offset地址处对应的int型字段的值，支持volatile语义
        var5 = this.getIntVolatile(var1, var2);
        
    // 拿期望值v与对象o的offset地址处的当前值比较，如果两个值相等，将当前值更新为v + delta，并返回true，否则返回false
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```




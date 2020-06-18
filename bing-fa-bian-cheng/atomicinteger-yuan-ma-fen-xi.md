# 1.AtomicInteger 源码分析

## 1.1.带着问题学习 {#TOP-%E5%B8%A6%E7%9D%80%E9%97%AE%E9%A2%98%E7%9C%8B%E6%BA%90%E7%A0%81}

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
// 获取 & 自增 原子地递增，并返回旧值
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
// 自增 & 获取 原子地递增，并返回新值
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
// 获取 & 自减 原子地递减，并返回旧值
public final int getAndDecrement() {
    return unsafe.getAndAddInt(this, valueOffset, -1);
}
// 自减 & 获取 原子地递减，并返回新值
public final int decrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, -1) - 1;
}
```

AtomicInteger 提供了自增/自减的两个场景方法，一个返回旧值，一个返回新增/自减后的。

实际都是通过Unsafe 的 getAndAddInt 方法来实现的，可以看到实际上 getAndAddInt 就是一个 cas + 自旋操作来实现。

```
   // 返回对象o的offset地址处的值，并将该值原子性地增加delta
    @HotSpotIntrinsicCandidate
    public final int getAndAddInt(Object o, long offset, int delta) {

        int v;

        do {
            // 获取对象o中offset地址处对应的int型字段的值，支持volatile语义
            v = getIntVolatile(o, offset);

            // 拿期望值v与对象o的offset地址处的当前值比较，如果两个值相等，将当前值更新为v + delta，并返回true，否则返回false
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));

        return v;
    }
```

```
    // 获取对象o中offset地址处对应的int型字段的值
    @HotSpotIntrinsicCandidate
    public native int getIntVolatile(Object o, long offset);

    // 拿期望值expected与对象o的offset地址处的当前值比较，如果两个值相等，将当前值更新为x
    @HotSpotIntrinsicCandidate
    public final boolean weakCompareAndSetInt(Object o, long offset, int expected, int x) {
        return compareAndSetInt(o, offset, expected, x);
    }
```

回到** 问题1,2**可以看到实际是采用 CAS + 自旋\(死循环\)来实现线程安全的自增。

# 2.总结

1.AtomicInteger 采用 CAS + 自旋\(死循环\)来实现线程安全的自增。

2.CAS是什么？

CAS全称为compare and swap，在使用上，通常会记录下某块内存中的旧值，通过对旧值进行一系列的操作后得到新值，然后通过CAS操作将新值与旧值进行交换。如果这块内存的值在这期间内没被修改过，则旧值会与内存中的数据相同，这时CAS操作将会成功执行 使内存中的数据变为新值。如果内存中的值在这期间内被修改过，则一般来说旧值会与内存中的数据不同，这时CAS操作将会失败，新值将不会被写入内存。


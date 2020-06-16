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






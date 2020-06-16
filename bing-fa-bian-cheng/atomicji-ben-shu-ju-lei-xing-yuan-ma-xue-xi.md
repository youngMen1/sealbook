# 1.Atomic基本数据类型源码学习

## 1.1.基本介绍

在并发编程中很容易出现并发安全的问题，有一个很简单的例子就是多线程更新变量i=1,比如多个线程执行i++操作，就有可能获取不到正确的值，而这个问题，最常用的方法是通过Synchronized进行控制来达到线程安全的目的（[关于synchronized可以看这篇文章](https://juejin.im/post/5ae6dc04f265da0ba351d3ff)）。但是由于synchronized是采用的是悲观锁策略，并不是特别高效的一种解决方案。实际上，在J.U.C下的atomic包提供了一系列的操作简单，性能高效，并能保证线程安全的类去更新基本类型变量，数组元素，引用类型以及更新对象中的字段类型。atomic包下的这些类都是采用的是乐观锁策略去原子更新数据，在java中则是使用CAS操作具体实现。

## 1.2.原子更新基本类型

Atomic包提高原子更新基本类型的工具类，主要有这些：

1. AtomicBoolean：以原子更新的方式更新boolean；
2. AtomicInteger：以原子更新的方式更新Integer;
3. AtomicLong：以原子更新的方式更新Long；

这几个类的用法基本一致，这里以AtomicInteger为例总结常用的方法

1. addAndGet\(int delta\) ：以原子方式将输入的数值与实例中原本的值相加，并返回最后的结果；
2. incrementAndGet\(\) ：以原子的方式将实例中的原值进行加1操作，并返回最终相加后的结果；
3. getAndSet\(int newValue\)：将实例中的值更新为新值，并返回旧值；
4. getAndIncrement\(\)：以原子的方式将实例中的原值加1，返回的是自增前的旧值；

还有一些方法，可以查看API，不再赘述。

# 2.怎么使用

为了能够弄懂AtomicInteger的实现原理，以getAndIncrement方法为例，来看下源码：

```
// This class intended to be implemented using VarHandles, but there are unresolved cyclic startup dependencies.

private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();

public final int getAndIncrement() {
    return U.getAndAddInt(this, valueOffset, 1);
}
```

可以看出，该方法实际上是调用了unsafe实例的getAndAddInt方法，unsafe实例的获取时通过UnSafe类的静态方法getUnsafe获取：

```
private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
```

Unsafe类在sun.misc包下，Unsafer类提供了一些底层操作，atomic包下的原子操作类的也主要是通过Unsafe类提供的compareAndSwapInt，compareAndSwapLong等一系列提供CAS操作的方法来进行实现。下面用一个简单的例子来说明AtomicInteger的用法：

```
/**
 * Atomic基本数据类型
 * @author fengzhiqiang
 * @date-time 2020/6/16 16:51
 **/
public class AtomicTest {
    
    private static AtomicInteger atomicInteger = new AtomicInteger(1);

    public static void main(String[] args) {
        System.out.println(atomicInteger.getAndIncrement());
        System.out.println(atomicInteger.get());
    }
}
```




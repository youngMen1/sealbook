# 1.Atomic基本数据类型源码学习

![](/static/image/微信截图_20200618103307.png)

## 1.1.基本介绍

```
// Integer类型（原子性）
public class AtomicInteger extends Number implements Serializable {
.......
}
```

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
输出结果：
1
2
```

例子很简单，就是新建了一个atomicInteger对象，而atomicInteger的构造方法也就是传入一个基本类型数据即可，对其进行了封装。对基本变量的操作比如自增，自减，相加，更新等操作，atomicInteger也提供了相应的方法进行这些操作。但是，因为atomicInteger借助了UnSafe提供的CAS操作能够保证数据更新的时候是线程安全的，并且由于CAS是采用乐观锁策略，因此，这种数据更新的方法也具有高效性。

AtomicLong的实现原理和AtomicInteger一致，只不过一个针对的是long变量，一个针对的是int变量。

而boolean变量的更新类AtomicBoolean类是怎样实现更新的呢?

核心方法是`compareAndSet`方法，其源码如下：

```
// 之前是这样写的

public final boolean compareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
    return unsafe.compareAndSwapInt(this, valueOffset, e, u);
}

// jdk8优化三元运算符了

// 如果当前值为expectedValue，则将其原子地更新为newValue，返回值表示是否更新成功
 public final boolean compareAndSet(boolean expectedValue, boolean newValue) {
        return VALUE.compareAndSet(this, (expectedValue ? 1 : 0), (newValue ? 1 : 0));
    }
```

可以看出，compareAndSet方法的实际上也是先转换成0,1的整型变量，然后是通过针对int型变量的原子更新方法compareAndSwapInt来实现的。可以看出atomic包中只提供了对boolean,int ,long这三种基本类型的原子更新的方法，参考对boolean更新的方式，原子更新char,doule,float也可以采用类似的思路进行实现。


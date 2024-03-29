# 1.JDK8的CAS实现学习笔记

## 1.1.带着问题学习

1.什么是CAS操作？

2.CAS的操作过程是怎样的？

## 1.2.基本介绍

CAS全称为compare and swap，是原子操作的一种，可用于在多线程编程中实现不被打断的数据交换操作，从而避免多线程同时改写某一数据时由于执行顺序不确定性以及中断的不可预知性产生的数据不一致问题。 该操作通过将内存中的值与指定数据进行比较，当数值一样时将内存中的数据替换为新的值。

在使用上，通常会记录下某块内存中的旧值，通过对旧值进行一系列的操作后得到新值，然后通过CAS操作将新值与旧值进行交换。如果这块内存的值在这期间内没被修改过，则旧值会与内存中的数据相同，这时CAS操作将会成功执行 使内存中的数据变为新值。如果内存中的值在这期间内被修改过，则一般来说旧值会与内存中的数据不同，这时CAS操作将会失败，新值将不会被写入内存。

## 1.3.**CAS的实现**

接下来我们去看CAS在java中的实现，[sun.misc.Unsafe](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/sun/misc/Unsafe.java)提供了compareAndSwap系列函数。

```
   /**
    * Atomically update Java variable to <tt>x</tt> if it is currently
    * holding <tt>expected</tt>.
    * @return <tt>true</tt> if successful
    */
   public final native boolean compareAndSwapObject(Object o, long offset,
                                                    Object expected,
                                                    Object x);

   /**
    * Atomically update Java variable to <tt>x</tt> if it is currently
    * holding <tt>expected</tt>.
    * @return <tt>true</tt> if successful
    */
   public final native boolean compareAndSwapInt(Object o, long offset,
                                                 int expected,
                                                 int x);

   /**
    * Atomically update Java variable to <tt>x</tt> if it is currently
    * holding <tt>expected</tt>.
    * @return <tt>true</tt> if successful
    */
   public final native boolean compareAndSwapLong(Object o, long offset,
                                                  long expected,
                                                  long x);
```

可以看到native发现这是一个本地方法调用，可以去查看对应的OpenJDK中调用代码[atomic\_linux\_x86.inline.hpp](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/os_cpu/linux_x86/vm/atomic_linux_x86.inline.hpp)/

[atomic\_windows\_x86.inline.hpp](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/os_cpu/windows_x86/vm/atomic_windows_x86.inline.hpp)

```
// linux(int 类型)
inline jint Atomic::cmpxchg(jint exchange_value, volatile jint* dest, jint compare_value) {
      int mp = os::is_MP();
      __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
      return exchange_value;
}

// windows(int 类型)
inline jint Atomic::cmpxchg(jint exchange_value, volatile jint* dest, jint compare_value) {
         return (*os::atomic_cmpxchg_func)(exchange_value, dest, compare_value);
}
```

# 2.**CAS在Java中的使用** {#CAS%E5%9C%A8Java%E4%B8%AD%E7%9A%84%E4%BD%BF%E7%94%A8}

如代码所示，compareAndSwapXx方法会根据第二个参数”偏移量”去拿偏移量这么多的属性的值和第三个参数对比，如果相同则将该属性值替换为第四个参数。该偏移量是指某个字段相对Java对象的起始位置的偏移量，可以通过unsafe.objectFieldOffset\(param\)去获取对应属性的偏移量。

```
/**
 * CAS在Java中的使用
 *
 * @author fengzhiqiang
 * @date-time 2020/6/16 16:20
 **/
public class UnsafeTest {

    private static Unsafe unsafe;

    static {
        try {
            // 通过反射获取rt.jar下的Unsafe类
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);
        } catch (Exception e) {
            System.out.println("Get Unsafe instance occur error" + e);
        }
    }

    public static void main(String[] args) throws Exception {
        Class clazz = Target.class;
        Field[] fields = clazz.getDeclaredFields();
        Target target = new Target();
        Field intFiled = clazz.getDeclaredField("intParam");
        Field longFiled = clazz.getDeclaredField("longParam");
        Field strFiled = clazz.getDeclaredField("strParam");
        Field strFiled2 = clazz.getDeclaredField("strParam2");

        // intParam
        System.out.print(unsafe.compareAndSwapInt(target, 12, 3, 10) + ":");
        System.out.println((Integer) intFiled.get(target));
        // longParam
        System.out.print(unsafe.compareAndSwapLong(target, 16, 1L, 2L) + ":");
        System.out.println((Long) longFiled.get(target));
        // strParam
        System.out.print(unsafe.compareAndSwapObject(target, 24, null, "5")
                + ":");
        System.out.println((String) strFiled.get(target));
        // strParam2
        System.out.print(unsafe.compareAndSwapObject(target, 28, null, "6")
                + ":");
        System.out.println((String) strFiled2.get(target));

    }

    static class Target {
        int intParam = 3;
        long longParam = 1L;
        String strParam;
        String strParam2;
    }
}
```

顺便介绍个查看对象的属性位置分布的一个小工具：[jol](http://openjdk.java.net/projects/code-tools/jol/)

首先引用jol-core包

```
<!-- https://mvnrepository.com/artifact/org.openjdk.jol/jol-core -->
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.9</version>
</dependency>
```

然后在项目里简单使用下：

```
public class TestOffset {
    public static void main(String[] args) {
        out.println(VM.current().details());
        out.println(ClassLayout.parseClass(Throwable.class).toPrintable());
    }
}
```

结果如下，根据偏移量界面化的显示属性分布的位置：

```
# Running 64-bit HotSpot VM.
# Using compressed oop with 3-bit shift.
# Using compressed klass with 3-bit shift.
# Objects are 8 bytes aligned.
# Field sizes by type: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]

java.lang.Throwable object internals:
 OFFSET  SIZE                            TYPE DESCRIPTION                               VALUE
      0    12                                 (object header)                           N/A
     12     4                                 (alignment/padding gap)                  
     16     4                java.lang.String Throwable.detailMessage                   N/A
     20     4             java.lang.Throwable Throwable.cause                           N/A
     24     4   java.lang.StackTraceElement[] Throwable.stackTrace                      N/A
     28     4                  java.util.List Throwable.suppressedExceptions            N/A
Instance size: 32 bytes
Space losses: 4 bytes internal + 0 bytes external = 4 bytes total
```

# 3.总结

## 3.1.什么是CAS操作

使用锁时，线程获取锁是一种悲观锁策略，即假设每一次执行临界区代码都会产生冲突，所以当前线程获取到锁的时候同时也会阻塞其他线程获取该锁。而CAS操作（又称为无锁操作）是一种乐观锁策略，它假设所有线程访问共享资源的时候不会出现冲突，既然不会出现冲突自然而然就不会阻塞其他线程的操作。因此，线程就不会出现阻塞停顿的状态。那么，如果出现冲突了怎么办？无锁操作是使用CAS\(compare and swap\)又叫做比较交换来鉴别线程是否出现冲突，出现冲突就重试当前操作直到没有冲突为止。

## 3.2.CAS的操作过程

CAS比较交换的过程可以通俗的理解为CAS\(V,O,N\)，包含三个值分别为：V 内存地址存放的实际值；O 预期的值（旧值）；N 更新的新值。当V和O相同时，也就是说旧值和内存中实际的值相同表明该值没有被其他线程更改过，即该旧值O就是目前来说最新的值了，自然而然可以将新值N赋值给V。反之，V和O不相同，表明该值已经被其他线程改过了则该旧值O不是最新版本的值了，所以不能将新值N赋给V，返回V即可。当多个线程使用CAS操作一个变量是，只有一个线程会成功，并成功更新，其余会失败。失败的线程会重新尝试，当然也可以选择挂起线程

CAS的实现需要硬件指令集的支撑，在JDK1.5后虚拟机才可以使用处理器提供的CMPXCHG指令实现。

# 3.3.**CAS存在问题** {#CAS%E5%AD%98%E5%9C%A8%E7%9A%84ABA%E9%97%AE%E9%A2%98}

1.CAS存的ABA问题

因为CAS会检查旧值有没有变化，这里存在这样一个有意思的问题。比如一个旧值A变为了成B，然后再变成A，刚好在做CAS时检查发现旧值并没有变化依然为A，但是实际上的确发生了变化。解决方案可以沿袭数据库中常用的乐观锁方式，添加一个版本号可以解决。原来的变化路径A-&gt;B-&gt;A就变成了1A-&gt;2B-&gt;3C。

2.自旋时间过长

使用CAS时非阻塞同步，也就是说不会将线程挂起，会自旋（无非就是一个死循环）进行下一次尝试，如果这里自旋时间过长对性能是很大的消耗。如果JVM能支持处理器提供的pause指令，那么在效率上会有一定的提升。


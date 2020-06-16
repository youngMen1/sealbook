# 1.JDK8的CAS实现学习笔记

## 1.1.TOP带着问题学习

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

# 3.总结

## 什么是CAS操作

使用锁时，线程获取锁是一种悲观锁策略，即假设每一次执行临界区代码都会产生冲突，所以当前线程获取到锁的时候同时也会阻塞其他线程获取该锁。而CAS操作（又称为无锁操作）是一种乐观锁策略，它假设所有线程访问共享资源的时候不会出现冲突，既然不会出现冲突自然而然就不会阻塞其他线程的操作。因此，线程就不会出现阻塞停顿的状态。那么，如果出现冲突了怎么办？无锁操作是使用CAS\(compare and swap\)又叫做比较交换来鉴别线程是否出现冲突，出现冲突就重试当前操作直到没有冲突为止。

## CAS的操作过程

CAS比较交换的过程可以通俗的理解为CAS\(V,O,N\)，包含三个值分别为：V 内存地址存放的实际值；O 预期的值（旧值）；N 更新的新值。当V和O相同时，也就是说旧值和内存中实际的值相同表明该值没有被其他线程更改过，即该旧值O就是目前来说最新的值了，自然而然可以将新值N赋值给V。反之，V和O不相同，表明该值已经被其他线程改过了则该旧值O不是最新版本的值了，所以不能将新值N赋给V，返回V即可。当多个线程使用CAS操作一个变量是，只有一个线程会成功，并成功更新，其余会失败。失败的线程会重新尝试，当然也可以选择挂起线程

CAS的实现需要硬件指令集的支撑，在JDK1.5后虚拟机才可以使用处理器提供的CMPXCHG指令实现。


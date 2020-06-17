# 1.Synchronized关键字分析

## 1.1.带着问题学习？

1.synchronized怎么使用的

2.synchronized底层是怎样实现了吗

## 1.2.基本介绍

相信大家都听说过线程安全问题，在学习操作系统的时候有一个知识点是**临界资源**，简单的说就是**一次只能让一个进程操作的资源**，但是我们在使用多线程的时候是并发操作的，并不能控制同时只对一个资源的访问和修改，想要控制那么有几种操作，今天我们就来讲讲第一种方法：线程同步块或者线程同步方法\(synchronized\);

根据Synchronized用的位置可以有这些使用场景：

![](/static/image/synchronized的使用场景.png)

如图，synchronized可以用在**方法**上也可以使用在**代码块**中，其中方法是实例方法和静态方法分别锁的是该类的实例对象和该类的对象。而使用在代码块中也可以分为三种，具体的可以看上面的表格。这里的需要注意的是：**如果锁的是类对象的话，尽管new多个实例对象，但他们仍然是属于同一个类依然会被锁住，即线程之间保证同步关系**。

## 1.3.synchronized实现原理

### 1.3.1.对象锁（monitor）机制

### 1.3.2.synchronized的happens-before关系

# 2.怎么使用

下面举一个例子说明synchronized关键字的使用

```
/**
 * Synchronized关键字源码分析
 *
 * @author fengzhiqiang
 * @date-time 2020/6/17 10:38
 **/
public class SynchronizedTest {
    /**
     * 未加synchronized输出的结果是没有顺序的
     * @param thread
     */
    public void insert(Thread thread) {
        for (int i = 0; i < 10; i++) {
            System.out.println(thread.getName() + "输出:  " + i);
        }
    }

    public static void main(String[] args) {
        final SynchronizedTest sychor = new SynchronizedTest();
        Thread t1 = new Thread() {
            @Override
            public void run() {
                sychor.insert(Thread.currentThread());
            }

            ;
        };
        Thread t2 = new Thread() {
            @Override
            public void run() {
                sychor.insert(Thread.currentThread());
            }

            ;
        };
        t1.start();
        t2.start();
    }
}
```

结果：

```
Thread-1输出:  0
Thread-1输出:  1
Thread-1输出:  2
Thread-1输出:  3
Thread-1输出:  4
Thread-1输出:  5
Thread-0输出:  0
Thread-1输出:  6
Thread-1输出:  7
Thread-0输出:  1
Thread-0输出:  2
Thread-0输出:  3
Thread-1输出:  8
Thread-0输出:  4
Thread-1输出:  9
Thread-0输出:  5
Thread-0输出:  6
Thread-0输出:  7
Thread-0输出:  8
Thread-0输出:  9
```

从上面的结果可以看出这里的两个线程是同时执行`insert()`方法的，输出的结果是没有顺序的

### 线程同步方法 {#%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5%E6%96%B9%E6%B3%95}

#### **下面我们在原有的代码上添加**`synchronized`**关键字看看效果如何**

```
/**
 * Synchronized关键字源码分析
 *
 * @author fengzhiqiang
 * @date-time 2020/6/17 10:38
 **/
public class SynchronizedTest {

    /**
     * 加了synchronized输出的结果是有顺序的
     * @param thread
     */
    public synchronized void insert(Thread thread) {
        for (int i = 0; i < 10; i++) {
            System.out.println(thread.getName() + "输出:  " + i);
        }
    }

    /**
     * 未加synchronized输出的结果是没有顺序的
     * @param thread
     */
//    public void insert(Thread thread) {
//        for (int i = 0; i < 10; i++) {
//            System.out.println(thread.getName() + "输出:  " + i);
//        }
//    }

    public static void main(String[] args) {
        final SynchronizedTest sychor = new SynchronizedTest();
        Thread t1 = new Thread() {
            @Override
            public void run() {
                sychor.insert(Thread.currentThread());
            }

            ;
        };
        Thread t2 = new Thread() {
            @Override
            public void run() {
                sychor.insert(Thread.currentThread());
            }

            ;
        };
        t1.start();
        t2.start();
    }
}
```

结果：

```
Thread-1输出:  0
Thread-1输出:  1
Thread-1输出:  2
Thread-1输出:  3
Thread-1输出:  4
Thread-1输出:  5
Thread-1输出:  6
Thread-1输出:  7
Thread-1输出:  8
Thread-1输出:  9
Thread-0输出:  0
Thread-0输出:  1
Thread-0输出:  2
Thread-0输出:  3
Thread-0输出:  4
Thread-0输出:  5
Thread-0输出:  6
Thread-0输出:  7
Thread-0输出:  8
Thread-0输出:  9
```

上面程序的运行结果我们可以看出加上了`synchronized`关键字使得线程是一个一个的执行的，只有先执行完一个线程才能执行了另外一个线程。

### 线程同步块

当然上面的我们使用的是线程同步方法，我们可以使用线程同步块，这两个相比线程同步块更加灵活，只需要将需要同步的代码放在同步块中即可，代码如下；

```
/**
 * Synchronized关键字源码分析
 *
 * @author fengzhiqiang
 * @date-time 2020/6/17 10:38
 **/
public class SynchronizedTest {

    public static void main(String[] args) {
        final SynchronizedTest sychor = new SynchronizedTest();
        Thread t1 = new Thread() {
            @Override
            public void run() {
                sychor.insert3(Thread.currentThread());
            }

            ;
        };
        Thread t2 = new Thread() {
            @Override
            public void run() {
                sychor.insert3(Thread.currentThread());
            }

            ;
        };
        t1.start();
        t2.start();
    }

    /**
     * 线程同步块
     *
     * @param thread
     */
    public void insert3(Thread thread) {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.println(thread.getName() + "输出:  " + i);
            }
        }
    }
}
```




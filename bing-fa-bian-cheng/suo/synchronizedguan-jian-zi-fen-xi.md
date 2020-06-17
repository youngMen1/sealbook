# 1.Synchronized关键字分析

## 1.1.带着问题学习？

1.synchronized怎么使用的

2.synchronized底层是怎样实现的

## 1.2.基本介绍

相信大家都听说过线程安全问题，在学习操作系统的时候有一个知识点是**临界资源**，简单的说就是**一次只能让一个进程操作的资源**，但是我们在使用多线程的时候是并发操作的，并不能控制同时只对一个资源的访问和修改，想要控制那么有几种操作，今天我们就来讲讲第一种方法：线程同步块或者线程同步方法\(synchronized\);

根据Synchronized用的位置可以有这些使用场景：

![](/static/image/synchronized的使用场景.png)

如图，synchronized可以用在**方法**上也可以使用在**代码块**中，其中方法是实例方法和静态方法分别锁的是该类的实例对象和该类的对象。而使用在代码块中也可以分为三种，具体的可以看上面的表格。这里的需要注意的是：**如果锁的是类对象的话，尽管new多个实例对象，但他们仍然是属于同一个类依然会被锁住，即线程之间保证同步关系**。

## 1.3.synchronized实现原理

### 1.3.1.对象锁（monitor）机制

### 1.3.2.synchronized的happens-before关系

### 1.3.3.锁获取和锁释放的内存语义

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
结果：
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
结果：
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

当然上面的我们使用的是线程同步方法，我们可以使用线程同步块，与这两个相比，线程同步块更加灵活，只需要将需要同步的代码放在同步块中即可，代码如下；

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

结果：
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

从上面的代码中可以看出这种方式更加灵活，只需要将需要同步的代码方法在同步块中，不需要同步的代码放在外面。

### 详细原因 {#%E8%AF%A6%E7%BB%86%E5%8E%9F%E5%9B%A0}

* 我们知道**每一个对象都有一把锁**，当我们使用线程同步方法或者线程同步块的时候实际上获得是对象的唯一的一把锁，当一个线程获得了这唯一的锁，那么其他的线程只能拒之门外了，**注意这里我们说是一个对象，也就是说是同一个对象，如果是不同的对象，那么就不起作用了，因为不同对象有不同的对象锁，**比如我们将上面的程序改成如下：

```
/**
 * @author fengzhiqiang
 * @date-time 2020/6/17 14:02
 **/
public class SynchronizedTest2 {

    /**
     * 此时线程同步块根本不起作用，因为他们调用的是不同对象的insert方法，获得锁是不一样的
     * @param args
     */
    public static void main(String[] args) {
        //第一个线程
        Thread t1 = new Thread() {
            @Override
            public void run() {
                SynchronizedTest2 sychor = new SynchronizedTest2();
                // 在run() 方法中创建一个对象
                sychor.insert(Thread.currentThread());
            }

            ;
        };
        // 第二个线程
        Thread t2 = new Thread() {
            @Override
            public void run() {
                SynchronizedTest2 sychor = new SynchronizedTest2();
                // 创建另外的一个对象
                sychor.insert(Thread.currentThread());
            }

            ;
        };
        t1.start();
        t2.start();
    }

    public void insert(Thread thread) {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.println(thread.getName() + "输出:  " + i);
            }
        }
    }

}
```

从上面的结果可知，此时线程同步块根本不起作用，因为他们调用的是**不同对象**的insert方法，获得锁是不一样的

* 上面我们已经说过一个对象有一把锁，线程同步方法和线程同步块实际获得的是对象的锁，因此线程同步块的括号中填入的是this，我们都知道this在一个类中的含义，**一个类也有唯一的一把锁**，我们前面说的是使用对象调用成员方法，现在如果我们要调用类中的静态方法，那么我们可以使用线程同步方法或者同步块获得类中的唯一一把锁，那么对于多个线程同时调用同一个类中的静态方法就可以实现控制了,代码如下:

```
/**
 * @author fengzhiqiang
 * @date-time 2020/6/17 14:14
 **/
public class SynchronizedTest3 {
    // 静态方法
    public static synchronized void insert(Thread thread) {
        for (int i = 0; i < 10; i++) {
            System.out.println(thread.getName() + "输出     " + i);
        }
    }

    public static void main(String[] args) {
        //第一个线
        Thread t1 = new Thread() {
            @Override
            public void run() {
                SynchronizedTest3.insert(Thread.currentThread());
                //直接使用类调用静态方法
            }

            ;
        };
        //第二个线程
        Thread t2 = new Thread() {
            @Override
            public void run() {
                SynchronizedTest3.insert(Thread.currentThread());
                //直接使用类调用静态方法
            }

            ;
        };
        t1.start();
        t2.start();
    }
}
```

#### 为静态方法或者静态方法中的代码块加上同步锁 {#%E4%B8%BA%E9%9D%99%E6%80%81%E6%96%B9%E6%B3%95%E6%88%96%E8%80%85%E9%9D%99%E6%80%81%E6%96%B9%E6%B3%95%E4%B8%AD%E7%9A%84%E4%BB%A3%E7%A0%81%E5%9D%97%E5%8A%A0%E4%B8%8A%E5%90%8C%E6%AD%A5%E9%94%81}

* 上面使用对象锁是为非静态方法实现线程同步的，**因为我们在调用非静态方法的时候需要创建对象，因此这里使用的是对象锁**。但是**我们调用静态方法的时候直接使用的是类名直接调用，并没有用到对象，因此我们需要用到类锁，**直接使用类名.class获取即可。

```
public class myThread{
    public static void display(){
        synchronized(myThread.class){
            for(int i=0;i<10;i++){
                System.out.println(i);
            }
        }
    }
}
```

# 3.总结

1. 要想实现线程安全和同步控制，如果执行的是非`static`同步方法或者其中的同步块，那么一定要使用同一个对象，如果调用的是static同步方法或者其中的同步块那么一定要使用同一个类去调用
2. 如果一个线程访问的是`static`同步方法，而另外一个线程访问的是非static的同步方法，此时这两个是不会发生冲突的，因为一个是类的锁，一个是对象的锁
3. 如果使用线程同步块，那么同步块中的代码是控制访问的，但是外面的代码是所有线程都可以访问的
4. 当一个正在执行同步代码块的线程出现了异常，那么`jvm`会自动释放当前线程所占用的锁，因此不会出现由于异常导致死锁的现象




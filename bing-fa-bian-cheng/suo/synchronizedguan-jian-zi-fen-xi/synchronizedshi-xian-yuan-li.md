```
1.synchronized实现原理
```

## 对象锁（monitor）机制

现在我们来看看synchronized的具体底层实现。先写一个简单的demo:

```
public class SynchronizedDemo {
        public static void main(String[] args) {
            synchronized (SynchronizedDemo.class) {
            }
            method();
        }

        private static void method() {
        }
    }
```

上面的代码中有一个同步代码块，锁住的是类对象，并且还有一个同步静态方法，锁住的依然是该类的类对象。编译之后，切换到SynchronizedDemo.class的同级目录之后，然后用**javap -v SynchronizedDemo.class**查看字节码文件：


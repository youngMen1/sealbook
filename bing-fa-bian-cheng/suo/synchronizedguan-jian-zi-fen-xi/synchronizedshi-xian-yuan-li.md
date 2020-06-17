# 1.synchronized实现原理

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




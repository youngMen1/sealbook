# 1.Atomic引用类型源码学习

## 1.1.基本介绍

如果需要原子更新引用类型变量的话，为了保证线程安全，atomic也提供了相关的类：

1. AtomicReference：原子更新引用类型；
2. AtomicReferenceFieldUpdater：原子更新引用类型里的字段；
3. AtomicMarkableReference：原子更新带有标记位的引用类型；

这几个类的使用方法也是基本一样的，以AtomicReference为例，来说明这些类的基本用法。下面是一个demo

```
public class AtomicTest {

    /**
     * 引用类型
     */
    private static AtomicReference<User> reference = new AtomicReference<>();

    public static void main(String[] args) {
        // 引用类型
        User user1 = new User("a", 1);
        reference.set(user1);
        User user2 = new User("b",2);
        User user = reference.getAndSet(user2);
        System.out.println(user);
        System.out.println(reference.get());
    }

    static class User {
        private String userName;
        private int age;

        public User(String userName, int age) {
            this.userName = userName;
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" +
                    "userName='" + userName + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}
输出结果：
User{userName='a', age=1}
User{userName='b', age=2}
```

首先将对象User1用AtomicReference进行封装，然后调用getAndSet方法，从结果可以看出，该方法会原子更新引用的user对象，变为`User{userName='b', age=2}`，返回的是原来的user对象User`{userName='a', age=1}`。


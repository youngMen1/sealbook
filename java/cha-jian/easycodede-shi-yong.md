# 1.基本介绍

Easycode是idea的一个插件，可以直接对数据的表生成entity,controller,service,dao,mapper,无需任何编码，简单而强大。

# 2.怎么使用

## 2.1.安装\(EasyCode\)

微信截图\_20200511093251.png

我这里的话是已经那装好了。

建议大家在安装一个插件，叫做Lombok。

Lombok能通过注解的方式，在编译时自动为属性生成构造器、getter/setter、equals、hashcode、toString方法。出现的神奇就是在源码中没有getter和setter方法，但是在编译生成的字节码文件中有getter和setter方法。

## 2.2.建立数据库

    -- ----------------------------
    -- Table structure for user
    -- ----------------------------
    DROP TABLE IF EXISTS `user`;
    CREATE TABLE `user` (
      `id` int(11) NOT NULL,
      `username` varchar(20) DEFAULT NULL,
      `sex` varchar(6) DEFAULT NULL,
      `birthday` date DEFAULT NULL,
      `address` varchar(20) DEFAULT NULL,
      `password` varchar(20) DEFAULT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    SET FOREIGN_KEY_CHECKS = 1;

## 2.3.在IDEA配置连接数据库

# 参考

[https://mp.weixin.qq.com/s/hLodJvBucYiz6BLrk0P5gQ](https://mp.weixin.qq.com/s/hLodJvBucYiz6BLrk0P5gQ)

EasyCode\(代码神器\)：

[https://www.jianshu.com/p/e4192d7c6844](https://www.jianshu.com/p/e4192d7c6844)


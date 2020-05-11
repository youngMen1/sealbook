# 1.基本介绍

Easycode是idea的一个插件，可以直接对数据的表生成entity,controller,service,dao,mapper,无需任何编码，简单而强大。

# 2.怎么使用

## 2.1.安装\(EasyCode\)

![img](/static/image/微信截图\_20200511093251.png)

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

在这个之前，新建一个Springboot项目，这个应该是比较简单的。

建好SpringBoot项目之后，如下图所示，找到这个Database

![img](/static/image/14226414-087e814930a90a37.webp)

按照如下图所示进行操作：

![img](/static/image/14226414-791374ad79eb0f53.webp)

然后填写数据库名字，用户名，密码。点击OK即可。这样的话，IDEA连接数据库就完事了

![img](/static/image/14226414-bf5916108c1d6c68.webp)

## 2.4.开始生成代码

在这个里面找到你想生成的表，然后右键，就会出现如下所示的截面。

![img](/static/image/14226414-9426d5d10eb698cb.webp)

点击1所示的位置，选择你要将生成的代码放入哪个文件夹中，选择完以后点击OK即可。

![img](/static/image/14226414-440d8fa96585bdaf.webp)

勾选你需要生成的代码，点击OK。

![img](/static/image/14226414-c16f4257fc98b322.webp)

这样的话就完成了代码的生成了，生成的代码如下图所示：

![img](/static/image/14226414-ca23bdf4c68cf497.webp)



# 参考

[https://mp.weixin.qq.com/s/hLodJvBucYiz6BLrk0P5gQ](https://mp.weixin.qq.com/s/hLodJvBucYiz6BLrk0P5gQ)

EasyCode\(代码神器\)：

[https://www.jianshu.com/p/e4192d7c6844](https://www.jianshu.com/p/e4192d7c6844)


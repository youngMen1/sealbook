# 1.基本介绍

Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化构建工具。它使用一种基于**Groovy的特定领域语言来声明项目设置，而不是传统的XML。**Gradle就是工程的管理，帮我们做了**依赖、打包、部署、发布、各种渠道的差异管理**等工作。

##### Gradle优势：

1. 一款最新的，功能最强大的构建工具，用它逼格更高
2. 使用程序代替传统的XML配置，项目构建更灵活
3. 丰富的第三方插件，让你随心所欲使用
4. Maven、Ant能做的，Gradle都能做，但是Gradle能做的，Maven、Ant不一定能做。

##### DSL介绍：

全称domain specific language，即特定领域语言  
有哪些常见的DSL语言：  
 xml、html  
DSL与通用编程语言的区别：  
 求专不求全，解决特定问题

##### Groovy介绍：

1. 一种基于JVM的敏捷开发语言
2. 结合了Python、Ruby和Smalltalk的许多强大的特性
3. Groovy可以与Java完美结合，而且可以使用Java所有的库
   ##### Groovy特性：
4. 语法上支持动态类型、闭包等新一代语言特性
5. 无缝集成所有已经存在的Java类库
6. 既支持面向对象编程也支持面向过程编程
   ##### Groovy优势：
7. 一种更加敏捷的编程语言
8. 入门非常的容易，且功能非常的强大
9. 既可以作为编程语言也可以作为脚本语言

# 2.IDEA中使用gradle

## 2.1.gradle下载

gradle[下载地址](https://services.gradle.org/distributions/)：`https://services.gradle.org/distributions/`

解压到  
![](/static/image/微信截图_20200612102610.png)

```
D:\utils\gradle-5.6.4
```

## 2.2.在IDEA里gradle配置和使用

打开环境配置，新建系统环境“GRADLE\_HOME”,值为

```
D:\utils\gradle-5.6.4
```

,找到path变量，后面添加

```
%GRADLE_HOME%\bin;
```

## 2.3.测试

在cmd命令里输入`gradle -v`如果能打出版本号，说明环境配置完毕。  
![](/static/image/微信截图_20200612102824.png)

## 2.4.IDEA配置

![](/static/image/微信截图_20200612102904.png)

## 2.5.构建命令

清理命令

```
gradle clean
```

构建打包命令

```
gradle clean build
```

编译时跳过测试，使用`-x`,`-x`参数用来排除不需要执行的任务

```
gradle clean build -x test
```

# 3.参考

idea中使用gradle：  
[https://www.cnblogs.com/liangzs/p/8855834.html](https://www.cnblogs.com/liangzs/p/8855834.html)


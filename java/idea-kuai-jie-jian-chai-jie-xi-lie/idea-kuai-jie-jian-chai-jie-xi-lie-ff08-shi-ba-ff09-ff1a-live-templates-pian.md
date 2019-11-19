这是IDEA快捷键拆解系列的第十八篇。

本文将介绍一下IDEA强大的Live Template功能。

首先，我们要知道Live Template是在哪里定义的，先按`Ctrl + Shift + S`进入设置，接着在输入框输入“Live Template”就可以定位到了，如下图所示。

从上图可以看到，IDEA官方已经帮我们定义好了一些常用的Live Template，而且针对不同的文件类型也划分了不同的Live Template Group。查看Live Template的快捷键是`Ctrl + J`，这里列几个比较常用的。

* iterate \(迭代\)

  * itar：Iterate elements of array，操作顺序迭代数组

    ```

    ```

  * ritar：Iterate elements of array in reverse order，反转迭代数组

    ```

    ```

  * iter：Iterate \(for each..in\)，ForEach迭代

    ```

    ```

  * fori：Create iteration loop，含下标的普通迭代

    ```

    ```

  * itli：Iterate elements of java.util.List，List迭代

    ```

    ```

  * itco：Iterate elements of java.util.Collection，iterator迭代

    ```

    ```

  * iten：Iterate java.util.Enumeration

  * itit：Iterate java.util.Iterator
  * ittok：Iterate tokens from String
  * itve：Iterate elements of java.util.Vector

* define \(定义\)

  * St
    ```

    ```
  * thr
    ```

    ```
  * psf
    ```

    ```
  * prsf
    ```

    ```
  * psfi
    ```

    ```
  * psfs
    ```

    ```
  * psfs
    ```

    ```
  * geti：Inserts singleton method getInstance
    ```

    ```
  * ifn：Inserts if null statement
    ```

    ```
  * inn：Inserts if not null statement
    ```

    ```
  * inst：Checks object type with instanceof and down-casts it

    ```

    ```

  * lazy：Performs lazy initialization

  * lst：Fetches last element of an array
  * mn：Sets lesser value to a variable
  * mx：Sets greater value to a variable
  * toar：Stores elements of java.util.Collection into array

* main

  * psvm
    ```

    ```

* print \(打印\)

  * sout：Prints a string to System.out
    ```

    ```
  * souf：Prints a formatted string to System.out
    ```

    ```
  * serr：Prints a string to System.err
    ```

    ```
  * soutm：Prints current class and method names to System.out
    ```

    ```
  * soutv：Prints a value to System.out
    ```

    ```
  * soutp：Prints method parameter names and values to System.out
    ```

    ```

* Maven

  * dep：dependency

    ```

    ```

  * pl：plugin

    ```

    ```

  * repo：repository

    ```

    ```

* SQL

  * col：new column definition

    ```

    ```

  * ins：insert rows into a table
    ```


    ```
  * sel：select all rows from a table
    ```

    ```
  * selc：select the number of specific rows in a table
    ```

    ```
  * selw：select specific rows from a table
    ```

    ```
  * tab：new table definition

    ```

    ```

  * upd：update values in a table
    ```

    ```

当然了，Live Templat强大的地方不仅在于官方定义好的这部分，更重要是还支持自定义的Live Template。在学习工作中，我们可以尝试把一些重复的代码抽出来变成模板，接下来我们就来看看如何自定义Live Template。

1. 配置自定义Live Template

2. 修改作用范围

K

1. 新建POJO，输入自定义的Live Template快捷键，如这里是cps，按
   `Enter`
   键选择后光标将停留在VAR1的位置，默认按
   `Tab`
   键即可快速切换到VAR2的位置。

> 由于个人疏忽，所以截图部分Live Template存在部分错误，下面是更正后的Live Template。

```
/**
 * $VAR1$
 */
private
String
 $VAR2$
;


$END$
```

作者：happyJared

链接：[https://www.jianshu.com/p/ee023c7f6241](https://www.jianshu.com/p/ee023c7f6241)

来源：简书

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


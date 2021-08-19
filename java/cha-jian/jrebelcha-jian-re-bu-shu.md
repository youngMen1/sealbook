# 1.JRebel插件热部署

包含：介绍jrebel、idea安装jrebel插件、激活jrebel\(非免费，需要免费激活使用\)、测试jrebel本地tomcat热部署、及解决jrebel插件不起作用

## 1.1.介绍

JRebel使你能即时分别看到代码、类和资源的变化，你可以一个个地上传而不是一次性全部部署。当程序员在开发环境中对任何一个类或者资源作出修改的时候，这个变化会直接反应在部署好的应用程序上，从而跳过了构建和部署的过程，每年可以省去部署用的时间花费高达5.25个星期。

JRebel是一款Java虚拟机插件，它使得我们能在不进行重部署的情况下，即时看到代码的改变对一个应用程序带来的影响。JRebel使你能即时分别看到代码、类和资源的变化，你可以一个个地上传而不是一次性全部部署。

## 1.2.安装JRebel

安装和使用JRebel需要注意两点：激活和设置

1、在IDEA中一次点击 File-&gt;Settings-&gt;Plugins-&gt;Brows Repositories  
 2、在搜索框中输入JRebel进行搜索  
 3、找到JRebel for intellij  
 4、install  
 5、安装好之后需要restart IDEA

![](/static/image/15645795-22ac925b9c130d7b.webp)

## 1.3.激活JRebel

JRebel并非免费的插件，需要激活之后才能使用。

[https://jrebel.qekang.com/](https://links.jianshu.com/go?to=https%3A%2F%2Fjrebel.qekang.com%2F){GUID}

最新激活url地址 : [http://139.199.89.239:1008/b8fdf475-b9f7-4146-b426-6e1bb5a17a16](http://139.199.89.239:1008/b8fdf475-b9f7-4146-b426-6e1bb5a17a16)  
下面的框中输入邮箱地址 , 可随意填 test@123.com. 然后点击右下角的激活按钮即可

在IDEA中一次点击 File-&gt;Settings-&gt;JRebel 并找到激活界面\(因为我的已经激活了，点击change liense进入的激活界面\)  
![](/static/image/15645795-beb15f99ca65f1b9.webp)  
![](/static/image/15645795-b1d7d1c6194267e9.webp)  
操作方法就是点击Work offile 按钮即可:  
![](/static/image/15645795-0359063b8432381a.webp)

# 2.总结

![](/static/image/微信截图_20200603113228.png)

如果出现激活过期的情况下 , 可以重新生成一下GUID , 替换原来的GUID即可 .  
在线生成GUID地址:

```
http://www.ofmonkey.com/transfer/guid
```

手动热部署：每次更改代码，不需要重启tomcat

编译代码

```
ctrl + F9
```

# 3.参考

IDEA JRebel插件热部署 史上最全：

[https://www.jianshu.com/p/2627e15d25a1](https://www.jianshu.com/p/2627e15d25a1)

JRebel 激活码 2020 亲测可用：

[https://www.2loveyou.com/articles/2020/01/09/1578533228431.html](https://www.2loveyou.com/articles/2020/01/09/1578533228431.html)

#### 生成 GUID 的网址 {#b3_solo_h4_0}

[https://www.guidgen.com/](https://www.guidgen.com/)

例如:

```
https://jrebel.qekang.com/738b776f-6cc9-4ac5-9574-960a057392db
```




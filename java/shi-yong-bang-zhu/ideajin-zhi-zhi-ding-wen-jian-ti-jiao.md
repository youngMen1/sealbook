# 1.IDEA禁止指定文件提交

## 1.1.idea svn提交时忽略.iml文件

**在File Types加**

```
idea;*.iml;
```

![](/static/image/微信截图_20210128142744.png)

**2020.3版本**  
![](/static/image/微信截图_20210129100449.png)

## 1.2.idea中git禁止提交隐藏指定文件

### 1.2.1.禁止提交指定文件（如iml后缀的文件）

**.ignor插件**

![](/static/image/20190126233842678.png)

### 1.2.2.禁止提交指定目录下的文件（如 .idea目录下的文件）

![](/static/image/20190126234003967.png)

### 1.2.3.IDEA提交时忽略某些文件的提交

### 1.2.4.intellij idea 忽略文件不提交

#### 创建一个changelist
首先创建一个changelist，为了好记，可以叫忽略的或者ignored，new changelist—-忽略的
![](/static/image/1313575-20181016090843893-2075448006.png)

将文件纳入ignored list
此时，修改了add.jsp，它会在Default里出现，如果我们不想提交，拖动文件到忽略的changelist
![](/static/image/微信截图_20210517151147.png)

只提交Default changelist
提交时，先点击Default，然后点击提交，就只提交指定的文件了。
![](/static/image/1313575-20181016090931858-658602583.png)

当然，在提交文件夹时，将某些文件排除后，会出现对话框，问你是否将排除的纳入另外一个changelist，那时候再建这个忽略的changelist也是可以的。

也可以通过文件夹分组
![](/static/image/1313575-20181016090943581-1255532069.png)

# 1.Linux 高级命令

像一些高级点的命令，比如说 Xargs 命令、管道命令、自动应答命令等，如果当初我要是知道，那我也可能写出简洁高效的脚本。

不管出于任何原因，我都想对一些 Linux 使用的高级命令进行用法说明，利人利己，以后不记得的话，我也可以回头翻来看看。

## 1.1.**实用的 xargs 命令**

**我们可以通过使用这个命令，将命令输出的结果作为参数传递给另一个命令。**

比如说我们想**找出某个路径下以 .conf 结尾的文件，并将这些文件进行分类**，那么普通的做法就是先将以 .conf 结尾的文件先找出来，然后输出到一个文件中，接着 cat 这个文件，并使用 file 文件分类命令去对输出的文件进行分类。

#### 例1：找出 / 目录下以 .conf 结尾的文件，并进行文件分类

#### 命令：find / -name \*.conf -type f -print \| xargs file

输出结果如下所示：

![](/static/image/640.webp)

xargs 后面不仅仅可以加文件分类的命令，你还可以加其他的很多命令，比如说实在一点的tar命令，**你可以使用find命令配合tar命令，将指定路径的特殊文件使用find命令找出来，然后配合tar命令将找出的文件直接打包**，命令如下：

```
find / -name *.conf -type f -print | xargs tar cjf test.tar.gz
```

## 1.2.**命令或脚本后台运行**

有时候我们进行一些操作的时候，不希望我们的操作在终端会话断了之后就跟着断了，特别是一些数据库导入导出操作，如果涉及到大数据量的操作，我们不可能保证我们的网络在我们的操作期间不出问题，所以后台运行脚本或者命令对我们来说是一大保障。

**比如说我们想把数据库的导出操作后台运行，并且将命令的操作输出记录到文件**，那么我们可以这么做：

```
nohup mysqldump -uroot -pxxxxx —all-databases > ./alldatabases.sql &（xxxxx是密码）
```

当然如果你不想密码明文，你还可以这么做：

```
nohup mysqldump -uroot -pxxxxx —all-databases > ./alldatabases.sql （后面不加&符号）
```

执行了上述命令后，会提示叫你输入密码，输入密码后，该命令还在前台运行，但是我们的目的是后天运行该命令，这个时候你可以按下Ctrl+Z，然后在输入bg就可以达到第一个命令的效果，让该命令后台运行，同时也可以让密码隐蔽输入。

命令后台执行的结果会在命令执行的当前目录下留下一个nohup.out文件，查看这个文件就知道命令有没有执行报错等信息。

## 1.3.**找出当前系统内存使用量较高的进程**

**在很多运维的时候，我们发现内存耗用较为严重，那么怎么样才能找出内存消耗的进程排序呢？**

```
ps -aux | sort -rnk 4 | head -20
```



  


# 2.参考

[https://mp.weixin.qq.com/s?\_\_biz=MzA4Nzg5Nzc5OA==∣=2651668001&idx=1&sn=5c147bba570c9a53862b6e3020d0f421&chksm=8bcbfd88bcbc749eb0cdd1f024dafc1fc51d6d0f233ca590d0c803b8fa2880ede75e9fb9dc83&scene=21\#wechat\_redirect](https://mp.weixin.qq.com/s?__biz=MzA4Nzg5Nzc5OA==&mid=2651668001&idx=1&sn=5c147bba570c9a53862b6e3020d0f421&chksm=8bcbfd88bcbc749eb0cdd1f024dafc1fc51d6d0f233ca590d0c803b8fa2880ede75e9fb9dc83&scene=21#wechat_redirect)


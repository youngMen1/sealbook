Linux是目前应用最广泛的服务器操作系统，基于Unix，开源免费，由于系统的稳定性和安全性，市场占有率很高，几乎成为程序代码运行的最佳系统环境。linux不仅可以长时间的运行我们编写的程序代码，还可以安装在各种计算机硬件设备中，如手机、路由器等，Android程序最底层就是运行在linux系统上的。

# 1.linux的目录结构

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPrCdhhPgOiaoBcFPce4tpjykKa6Mib0INI8WR69mJnFnqycibJJyudibgmAPfx8eDgL3p9Phe3tgFmVzA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
                                                  / 下级目录结构
```

* bin \(binaries\)存放二进制可执行文件

* sbin \(super user binaries\)存放二进制可执行文件，只有root才能访问

* etc \(etcetera\)存放系统配置文件

* usr \(unix shared resources\)用于存放共享的系统资源

* home 存放用户文件的根目录

* root 超级用户目录

* dev \(devices\)用于存放设备文件

* lib \(library\)存放跟文件系统中的程序运行所需要的共享库及内核模块

* mnt \(mount\)系统管理员安装临时文件系统的安装点

* boot 存放用于系统引导时使用的各种文件

* tmp \(temporary\)用于存放各种临时文件

* var \(variable\)用于存放运行时需要改变数据的文件

# 2.linux常用命令

**命令格式**：命令 -选项 参数 （选项和参数可以为空）

```
如：ls -la /usr
```

#### 2.1 操作文件及目录

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiaMyzWUmHXfOmbOtzqmF8XFrlGTZjXlldGIjFIzXjjVd9qqfv5DNZe4Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiaxBcpuWAufxawefmBjR3svV0XfIDe91ANDN7POcialBPIRX4UCVdHmew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiaL2xfnaDDv2bGznogE1ppXPxha6U6hQ5aYP9evavx9Aw3D8Tl5LuMWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 2.2 系统常用命令

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiaykCib7bstwWapYzXfQtvlZPPVdLpyB61yoIh62Ss300C5eYRTeObMzQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiaFv8ibw718CI6CQMZoiaXFsGLm776H7a9xfHiatLOfGfhxw4GQjia4lBlPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 2.3 压缩解压缩

#### ![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiayWYRKtESM2QO2PibplMMtwbtlWDV5GD7y0BlP03xyUJmrkkZIBmbwkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)2.4 文件权限操作

* linux文件权限的描述格式解读

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPrCdhhPgOiaoBcFPce4tpjykvNabx6Ho22UJSicXj6uGwrib48KAjv8WTn0iaaOUBSLYMUsKiaDe8vb7gA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

* r 可读权限，w可写权限，x可执行权限（也可以用二进制表示 111 110 100 --&gt; 764）

* 第1位：文件类型（d 目录，- 普通文件，l 链接文件）

* 第2-4位：所属用户权限，用u（user）表示

* 第5-7位：所属组权限，用g（group）表示

* 第8-10位：其他用户权限，用o（other）表示

* 第2-10位：表示所有的权限，用a（all）表示

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiaJYiauCAFjempsSiaUsNEQKU5muRNamqCt6XKxX51uUcUJQIfauCBGb7g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 3.linux系统常用快捷键及符号命令

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiaF8T3HwDibZb6EoGzCuTP6KGkCtByUcHRfpWL288lJPFkNQjicYWdnKoQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 4.vim编辑器

vi / vim是Linux上最常用的文本编辑器而且功能非常强大。只有命令，没有菜单，下图表示vi命令的各种模式的切换图。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tuSaKc6SfPrCdhhPgOiaoBcFPce4tpjykeSGAqQ6vwicJcNf9AtHRpxAb8efawNibqgx2zmkCReVbLYSCuyvXIM6w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 4.1 修改文本

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiapsRoicnGDROYs7K2Nby9K0cPTQwvicia5pg58pPFRbeYQX23bUriaP9c3Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

:q! 不保存退出

#### 4.2 定位命令

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiaW9ubIw2eADU27yyNHrgc3qUnjicHWCGV13iaCAcaegLtEXlujR2gyz4w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 4.3 替换和取消命令

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCialCmRBeFam8sY6ibjib0qx75hBPrISqQwywtjq4LZESCD7JYwnmuJvhgg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 4.4 删除命令

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCiaJddPfgVP6wh2MZB6Rx7gLLtofaBP6s7j1Ao12RKJcvzZt4IWMHIGdw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 4.5 常用快捷键

![](https://mmbiz.qpic.cn/mmbiz_png/YrLz7nDONjFfwIxuqgHaxR6TVoWwicBCia5JB4PkqGw6Qo1icQsc57GaxNUF7AuSzEULiaBomtvcnuhAAicCtpdhqzQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

---

# 5.总结:

### 1.今天，无意发现一个很简单的办法，可以直接让jps显示出真实的jar包名称，简单到哭，说出来都没人信。

java -jar  jar包的完整路径

```
比如： java  -jar  /home/weblogic/test/hello.jar
```

## 2.查看.tar.gz文件内容（不需要解压）

### **zcat命令：**用于不真正解压缩文件，就能显示压缩包中文件的内容的场合。

服务器上的日志大多数都是对几天前的日志进行tar.gz压缩（例如：7天前的日志），而有的时候我们需要查看历史日志，且又不想解压该日志，这时，我们可以使用下面的方法实现：

```
zcat information-rest-ms.log.2020-05-07.0.gz
zcat gpush-sms.log.2020-05-14.0.gz
```

如果有需要进行过滤的需求，可以使用下面的方式实现：

```
zcat information-rest-ms.log.2020-05-07.0.gz  | grep -a -C30 7494560570
或者
zcat information-rest-ms.log.2020-05-07.0.gz  | grep '7494560570'
```

### **根据关键字或日期查找日志:**

1.VI:单个文件可以使用vi或vim编辑器打开日志文件，使用编辑器里的查找功能。在查看模式下，符号/后面跟关键字向下查找，符号?后面跟关键字向上查找，按n查找下一个，按N查找上一个。

```
vim gpush-sms.log
```

2.grep命令：cat 1.log \| grep key  可以写为： grep key 1.log

根据字符串查询日志中关键词出现的位置：cat -n 日志文件\| grep 'keyword'

例：cat -n 1.log \| grep 'keyword'

```
cat goods-service.log|grep '调用vip-service服务验证是否为会员'
cat gpush-sms.log|grep '短信发送失败'
```

检索日志，并显示该条日志的前后N（10）行记录：cat 日志文件 \| grep -n -B10 -A10 "关键字"

3.查看某段时间内的日志： sed -n '/起始时间/,/结束时间/p' 日志文件，

查看某段时间内的关键字日志：sed -n '/起始时间/,/结束时间/p' 日志文件\| grep ‘keyword’

```
例：sed -n ‘/2018-06-21 14:30:20/,/2018-06-21 16:12:00/p’ catalina.out | grep ‘keyword’
```

4.

tail  -n  10  日志文件   查询日志尾部最后10行的日志;

tail -n +10 日志文件    查询10行之后的所有日志;

head -n 10  日志文件  查询日志文件中的头10行日志;

head -n -10  日志文件  查询日志文件除了最后10行的其他所有日志;

![img](/static/image/微信截图_20200515163815.png)

# 6.参考:

原文链接：[https://blog.csdn.net/lch\_2016/article/details/81334993](https://blog.csdn.net/lch_2016/article/details/81334993)

[https://mp.weixin.qq.com/s/fmhBcMugMhNlBu\_PEhAk2A](https://mp.weixin.qq.com/s/fmhBcMugMhNlBu_PEhAk2A)
Linux命令大全:https://linux265.com/course/linux-command-ab.html

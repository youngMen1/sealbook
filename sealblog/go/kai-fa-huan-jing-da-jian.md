# 1.开发环境搭建

## 1.1.下载并且安装/解压Golang

```
https://studygolang.com/dl
```

![](/static/image/1599027458.jpg)

## 2.环境变量配置

windows需要配置几个环境变量，我是下载的压缩文件，所以需要自己配置，通过安装程序安装的应该不需要设置环境变量，也可以去看下环境变量是否设置起。

  
![](/static/image/1599027545.jpg)

GOBIN 是安装go目录里面的bin文件夹  
GOPATH是你的工作目录  
GOROOT是安装go的根目录

另外需要把GOBIN加入到path里面

到这里go的基本配置就ok了，另外需要注意的是，你的go项目文件，都是放在工作目录里面的src下面的，一个项目一个文件。

![](/static/image/1599027767.jpg)

## 3.go基本命令


```
https://blog.csdn.net/y472360651/article/details/82914263
```


go语言自带一套完整的命令操作系统，你可以在终端中执行go命令来进行查看！接下来介绍一些常用的go命令操作


`go version`
查看go当前的版本

`go env`
查看当前go的环境变量

`go list`
查看当前安装的全部package

`go run`
编译并运行go程序

`go build`
这个命令主要用于编译代码。在包的编译过程中，若有必要，会同时编译与之相关联的包。go build命令有如下几点值得注意的地方：

如果是普通包，当你执行go build命令之后，它不会产生任何文件。如果你需要在$GOPATH/pkg下生成相应的文件，那就得执行go install命令
如果是main包，当你执行go build命令之后，它就会在当前目录下生成一个可执行文件。如果你需要在$GOPATH/bin下生成相应的文件，那么你需要执行go install命令，或者使用go build -o 路径/xx
如果项目文件夹中有多个文件，但是此时你只想编译其中某一个文件，就可以在go build之后加上文件名，例如：go build demo_1.go。默认情况下，build命令会编译当前目录下所有go文件
你也可以指定编译输出的文件名。我们可以使用go build -o xxx，来指定编译输出的文件名，默认情况下是你的项目名
go build会自动忽略以_或者.开头的go文件
如果你的源代码针对不同操作系统需要不同的处理，那么你可以根据不同操作系统后缀来命名文件。例如有一个读取数组的程序，它对于不同的操作系统可能有如下几个源文件：


```
array_linux.go
array_windows.go
array_darwin.go
1
2
3
```


go build会选择性的编译以系统名结尾的文件，例如，在Linux系统下编译只会选择array_linux.go文件，其他系统命名后缀文件全部忽略！

**常用参数介绍：**

-o：指定输出的文件名，可以带上路径，例如：`go build -o app/a.exe`
-i：安装相应的包，相当于build+install
-a：更新全部已经是最新的包的


-n：把需要执行的编译命令打印出来，但是不执行它们
-p n：指定可以并行运行的编译数目，默认是CPU数量
-race：开启编译的时候自动检测数据竞争的情况，目前只支持64位的机器
-v：打印出来正在编译的包名
-work：打印出来编译时间的临时文件夹名称，并且如果已经存在的话就不要删除
-x：打印出来执行的命令，与-n类似，只不过这个会执行
go clean
这个命令是用来移除当前源码包和关联源码包里面编译生成的文件，这些文件包括：

_obj/：旧的object目录，由Makefiles遗留
_test/：旧的test目录，由Makefiles遗留
_testmain.go：旧的gotest文件，由Makefiles遗留
test.out：旧的test记录，由Makefiles遗留
build.out：旧的test记录，由Makefiles遗留
*.[568ao]：object文件，由Makefiles遗留
DIR(.exe)：由go build产生
DIR.test(.exe)：由go test -c产生
MAINFILE(.exe)：由go build MAINFILE.go产生
*.so：由SWIG产生

**参数介绍：**

-i：清楚关联的安装包和可运行文件，也就是通过go install安装的文件
-n：把需要执行的清除命令打印出来，但是不执行
-f：循环的清除在import引入的包
-x：打印出来执行的详细命令，与-n类似，只不过这个会执行
举个栗子：



```
$ go clean -i -x
cd /root/go_path/src/cx_demo
rm -f cx_demo cx_demo.exe cx_demo.test cx_demo.test.exe demo_1 demo_1.exe
rm -f /root/go_path/bin/cx_demo
1
2
3
4
go fmt
```


在go中，代码有着标准的风格。由于之前已经有的一些习惯或其它原因我们常将代码写成ANSI风格或者其它更适合自己的格式，这将为人们在阅读别人的代码时添加不必要的负担，所以go强制了代码格式(比如大括号必须放在行尾)，不按照此排版的代码将不能编译通过，为了减少浪费在排版上的时间，go工具集中提供了一个go fmt命令，它可以帮你格式化你写好的代码文件。

参数介绍：
-n：标记打印将要执行的命令
-x：打印出来执行的详细命令，与-n类似，只不过这个会执行
go get
这个命令是用来获取远程代码包的，目前支持的有BitBucket、GitHub、Google Code和LaunchPad。这个操作实际上在内部分为了两步：第一步是下载源码包，第二部是执行go install。下载源码包的go工具会自动根据不同的域名调用不同的源码工具，对应关系如下：



```
BitBucket(Mercurial Git)
GitHub(Git)
Google Code Project Hosting(Git, Mercurial, Subversion)
LaunchPad(Bazaar)
1
2
3
4
```


所以，为了go get能正常工作，你必须安装了合适的源码管理工具，并同时把这些命令加入你的PATH中。

**参数介绍：**

-d：只下载不install
-f：只有设置了-u参数才有意义！不让-u去验证import中的每一个都已经获取了
-fix：在获取源码之后先运行fix，然后再去做其他的事情
-t：同时下载需要为运行测试所需要的包
-u：强制使用网络去更新包和它的依赖
-v：显示执行的命令


```

go install
```



这个命令在内部实际分成了两步操作：第一步是生成结果文件(可执行文件或者.a包)，第二部会把编译好的结果转移到$GOPATH/bin或者$GOPATH/pkg。参数支持所有的go build参数，但是，我们只需要记住一个-v参数即可，这可以可以随时查看底层的执行信息！值得注意的是，有两种方式进行安装：

进入对应的项目目录之下，然后执行go isntall，就可以安装了
在任意的目录之下执行go install xxx
go test
执行这个命令，会自动读取源码目录下面的*_test.go文件，生成并运行测试用的可执行文件。输出的信息类似：



```
ok   archive/tar   0.011s
FAIL archive/zip   0.022s
ok   compress/gzip 0.033s
...
1
2
```



## 4.go语言开发
![](/static/image/1599028017.jpg)





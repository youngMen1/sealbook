# 1.什么是Protobuffer

## 1.1.Protocol Buffers简介

Protocol Buffers \(ProtocolBuffer/ protobuf \)是Google公司开发的一种数据描述语言，它独立于语言，独立于平台。类似于XML能够将结构化数据序列化，可用于数据存储、通信协议等方面。现阶段支持C++、JAVA、Python等三种编程语言。

ProtoBuf是Google开源的一套二进制流网络传输协议，它独立于语言，独立于平台。google 提供了多种语言的实现：**java、C\#、C++、**[**Go**](http://lib.csdn.net/base/go)** 和**[**Python**](http://lib.csdn.net/base/python)，每一种实现都包含了相应语言的编译器以及库文件。由于它是一种二进制的格式，比使用 xml 进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域。

优点：传输效率快（比xml和json快10-20倍），文档型协议；  
缺点：使用不太方便；

这里简单解释一下什么是文档型协议，向我们的xml和json一般在使用的时候都需要保存一份说明文档和一个实际的java类，而protobuf在使用的时候其定义的格式就是说明文档，简单明了而且可以将其编译成各个平台的类库，以java平台为例，其编程成jar之后，若定义文件发生了变化，则在使用jar包的话就会报错，必须重新编译，这也就保证了App端与服务器端的协议统一性。

## 1.2.Protobuffer相比Xml的优点

* 更简单，高效

* 跨平台，跨语言

* 向下兼容，易升级

* 代码自动生成，不用手写

* 生成了更容易在编程中使用的数据访问类

## 1.3.数据交互xml、json、protobuf格式比较

1、json: 一般的web项目中，最流行的主要还是json。因为浏览器对于json数据支持非常好，有很多内建的函数支持。

2、xml: 在webservice中应用最为广泛，但是相比于json，它的数据更加冗余，因为需要成对的闭合标签。json使用了键值对的方式，不仅压缩了一定的数据空间，同时也具有可读性。

3、protobuf:是后起之秀，是谷歌开源的一种数据格式，适合高性能，对响应速度有要求的数据传输场景。因为profobuf是二进制数据格式，需要编码和解码。数据本身不具有可读性。因此只能反序列化之后得到真正可读的数据。

相对于其它protobuf更具有优势

1：序列化后体积相比Json和XML很小，适合网络传输

2：支持跨平台多语言

3：消息格式升级和兼容性还不错

4：序列化反序列化速度很快，快于Json的处理速度

结论：

在一个需要大量的数据传输的场景中，如果数据量很大，那么选择protobuf可以明显的减少数据量，减少网络IO，从而减少网络传输所消耗的时间。

# 2.怎么使用

## 2.1安装环境

![img](/static/image/微信截图_20200418181622.png)

## 2.2.windows环境下的安装与使用：

### 2.2.1.protocol编译器安装

**安装protocol编译器，用来编译.proto文件。**

##### 2.2.1.1.下载地址：

```
// 里面有windows版的:protoc-3.11.1.win32.zip
https://github.com/google/protobuf/releases
```

![img](/static/image/微信截图_20200418175815.png)

##### 2.2.2.2.安装。

下载完解压后，如果不想安装，可直接在cmd窗口进入解压得到的bin目录操作。

安装，把bin目录copy下来，放到操作系统环境变量的path变量后面。  
![img](/static/image/微信截图_20200418175704.png)

然后就可以在cmd中输入protoc,他应该会像配置Java环境那样有一大长串说明安装成功  
![img](/static/image/微信截图_20200418175606.png)

### 2.2.2.编译

#### 2.2.2.1引入依赖

```
 <!--生成器的版本和maven版本要一致-->
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
        <!--<version>3.6.0</version>-->
            <version>3.11.4</version>
        </dependency>
```

#### 2.2.2.2.编译命令

* protoc，编译命令；

* --proto\_path,就是你的proto文件所在目录是哪。我这里是D:\protoc-3.11.4-win64\bin。

* --java\_out，标识输出的java文件应该放在哪个目录。这里的 ./ 是指当前目录。

* ProtoDemo.proto，就是我们要编译的文件。

```
protoc.exe --java_out=./ ProtoDemo.proto
或者
protoc --protopath D:\protoc-3.11.4-win64\bin --java_out ./ ProtoDemo.proto
```

#### 2.2.2.3.例子

新建一个ProtoDemo.proto文件

```
syntax = "proto2";
package com.seal.protobuf.msg;
option java_package = "com.seal.protobuf.msg";
option java_outer_classname = "ProtoDemo";
message demo{
    required int32 id = 1;
    required string name = 2;
    optional string email = 3;
    repeated string friends = 4;
    
}
```

# 3.总结

下载地址：[https://github.com/google/protobuf/releases](https://github.com/google/protobuf/releases。里面有windows版的:protoc-3.6.1.win32.zip。)

![img](/static/image/微信截图_20200420101711.png)

![img](/static/image/微信截图_20200418181622.png)

![img](/static/image/微信截图_20200420101353.png)

![img](/static/image/微信截图_20200420102349.png)

```
protoc.exe --java_out=./ PersonBean.proto
```

可以看到，控制台中打印了客户端发送过去的Person消息，这样，一个使用ProtocolBuffer来进行数据发送和接收的java程序就完成了

## 3.1protoc.exe如何使用

直接上命令：（datagram.proto 文件和protoc.exe同一目录下，如果不同-I=后面为相应的路径）

```
#c++的命令
protoc.exe -I=. --cpp_out=./  datagram.proto 

#java的命令
protoc.exe -I=. --java_out=./ datagram.proto

#其他类推
```

# 4.参考

源码:

[https://github.com/protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf)

Java中使用Protocol Buffer:  
[https://www.cnblogs.com/blythe/articles/8473016.html](https://www.cnblogs.com/blythe/articles/8473016.html)

protobuf 3.5 java使用介绍:  
[https://blog.csdn.net/fangxiaoji/article/details/78826165](https://blog.csdn.net/fangxiaoji/article/details/78826165)

[windows环境下protobuf的java操作{编译，序列化，反序列化}](https://www.cnblogs.com/InformationGod/p/9479919.html)

[https://www.cnblogs.com/InformationGod/p/9479919.html](https://www.cnblogs.com/InformationGod/p/9479919.html)

Protobuf3语言指南:

[https://blog.csdn.net/u011518120/article/details/54604615](https://blog.csdn.net/u011518120/article/details/54604615)

ET篇:Google.Protobuf的学习\(二:使用protoc.exe生成自己的类\)

[https://www.pianshen.com/article/3303219954/](https://www.pianshen.com/article/3303219954/)

Cannot resolve symbol 'UnusedPrivateParameter'：

[https://blog.csdn.net/dataiyangu/article/details/105183040](https://blog.csdn.net/dataiyangu/article/details/105183040)

[Java中使用Protocol Buffer](https://www.cnblogs.com/blythe/articles/8473016.html)例子:

[https://www.cnblogs.com/blythe/articles/8473016.html](https://www.cnblogs.com/blythe/articles/8473016.html)


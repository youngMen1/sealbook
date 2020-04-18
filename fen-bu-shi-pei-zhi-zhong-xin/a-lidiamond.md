# 1.基本介绍

**阿里巴巴集团开源的框架分布式配置中心diamond**

diamond是一个管理持久配置（持久配置是指配置数据会持久化到磁盘和数据库中）的系统。

无可厚非，淘宝内部正在使用diamond，在淘宝内部的绝大多数系统的配置都是由diamond统一管理的。

diamond最大的特点就是简单、可靠、易用。diamond的简单是指diamond整体结构非常简单，从而减少了出错的可能性；

diamond的可靠是指应用方在任何情况下都可以启动，

例如：淘宝的核心系统最初一年多是由diamond所管理，在这期间并没有发生什么大的故障；diamond的易用是指客户端使用只需要两行代码，暴露出的接口都非常简单，易于理解。

对于应用系统而言，diamond为其提供获取配置的服务，应用不仅可以在启动时从diamond获取相关的配置，而且可以在运行中对配置数据的变化进行感知并获取变化后的配置数据。

## 1.1.源代码检出

从以下svn地址检出diamond的源代码：

[http://code.taobao.org/svn/diamond/trunk](http://code.taobao.org/svn/diamond/trunk)

## 1.2.server的搭建

完成后，请将数据库的配置信息（IP，用户名，密码）添加到diamond-server工程的src/resources/jdbc.properties文件中的db.url，db.user，db.password属性上面，这里建立的库名，用户名和密码，必须和jdbc.properties中对应的属性相同。

### 1.2.1.mysql

安装mysql-server的步骤请参考mysql官方文档，安装完毕后，建立数据库，然后建立两张表，建表语句分别如下：

    create table config_info (

    `id` bigint(64) unsigned NOT NULL auto_increment,

    `data_id` varchar(255) NOT NULL default ’ ’,

    `group_id` varchar(128) NOT NULL default ’ ’,

    `content` longtext NOT NULL,

    `md5` varchar(32) NOT NULL default ’ ’,

    `src_ip` varchar(20) default NULL,

    `src_user` varchar(20) default NULL,

    `gmt_create` datetime NOT NULL default ’2010-05-05 00:00:00′,

    `gmt_modified` datetime NOT NULL default ’2010-05-05 00:00:00′,

    PRIMARY KEY  (`id`),

    UNIQUE KEY `uk_config_datagroup` (`data_id`,`group_id`)

    );

    create table group_info (

    `id` bigint(64) unsigned NOT NULL auto_increment,

    `address` varchar(70) NOT NULL default ’ ’,

    `data_id` varchar(255) NOT NULL default ’ ’,

    `group_id` varchar(128) NOT NULL default ’ ’,

    `src_ip` varchar(20) default NULL,

    `src_user` varchar(20) default NULL,

    `gmt_create` datetime NOT NULL default ’2010-05-05 00:00:00′,

    `gmt_modified` datetime NOT NULL default ’2010-05-05 00:00:00′,

    PRIMARY KEY  (`id`),

    UNIQUE KEY `uk_group_address` (`address`,`data_id`,`group_id`)

    );

建表完成后，请将数据库的配置信息添加到diamond-server工程的src/resources/jdbc.properties文件中。

**注意:**

完成后，请将数据库的配置信息（IP，用户名，密码）添加到diamond-server工程的src/resources/jdbc.properties文件中的db.url，db.user，db.password属性上面，这里建立的库名，用户名和密码，必须和jdbc.properties中对应的属性相同。

### 1.2.2.tomcat

tomcat是diamond server的运行容器。

tomcat的安装请参考tomcat官方文档，建议使用tomcat7

不需要对tomcat进行任何改动。

### 1.2.3.diamond server

在diamond-server源代码根目录下，执行mvn clean package -Dmaven.test.skip，成功后会在diamond-server/target目录下生成diamond-server.war（如果没有安装maven，请参考maven官方文档进行安装）。

打包完成后，将diamond-server.war放在tomcat的webapps目录下。启动tomcat，即启动了diamond-server。

### 1.2.4.http server

http server用来存放diamond server等地址列表，可以选用任何http server，这里以tomcat为例。一般来讲，http server和diamond server是部署在不同机器上的，这里简单起见，将二者部署在同一个机器下的同一个tomcat的同一个应用中，注意，如果部署在不同的tomcat中，端口号一定是8080，不能修改（所以必须部署在不同的机器上）。在上一步的tomcat的webapps中的diamond-server中建立文件diamond，文件内容是diamond-server的地址列表，一行一个地址，地址为IP，例如：127.0.0.1。完成以上4步后，server端的搭建就完成了。

### 1.2.5.pushit

pushit是一个轻量级的消息通知服务组件，用来为diamond做实时通知服务，通知客户端数据的变化，它也是CS的结构，服务端搭建步骤如下：

在pushit源代码根目录下，执行mvn clean package assembly:assembly -Dmaven.test.skip命令，成功后会在pushit/target目录中看到pushit-pushit.tar.gz包。

执行tar -xzvf  pushit-pushit.tar.gz，进行解压。

进入pushit目录，建立logs目录，在logs目录中建立pushit.log文件。

进入pushit/bin目录，执行./pushit-startup.sh ../conf/server.properties命令，启动pushit-server

### 1.2.6.redis

redis用来存放一些跟统计相关的信息。

redis的安装请参考redis的官方文档。

安装完成后，请在diamond-server的配置文件redis.properties中填写对应的信息。

完成以上6步后，server端的搭建就完成了。

## 1.3.发布数据

diamond发布数据通过手工的方式进行。

修改diamond-server的配置文件user.properties，以k=v的方式添加登录diamond-server的用户名和密码。

在浏览器中输入[http://ip:port/diamond-server/，ip和port为server搭建的第（2）步中的地址和端口，登录后进入后台管理界面，然后点击“配置信息管理”——](http://ip:port/diamond-server/，ip和port为server搭建的第（2）步中的地址和端口，登录后进入后台管理界面，然后点击“配置信息管理”——) “添加配置信息”，在输入框中输入dataId、group、内容，最后点击“提交”即可。

成功后，可以在“配置信息管理”中查询到发布的数据。

## 1.4.订阅数据

diamond客户端API主要提供了订阅数据的功能.

### 1.4.1.客户端获取服务端地址

获取服务端地址对客户端是透明的，客户端仅仅需要在本地进行如下域名绑定即可：

domain  ip

其中，domain的值与diamond-utils工程下的com.taobao.diamond.common.Constants类中的DEFAULT\_DOMAINNAME和DAILY\_DOMAINNAME的值相同，ip为server搭建第（4）步中的http server地址。

### 1.4.2.创建订阅者

```
DiamondManager manager = new DefaultDiamondManager(group, dataId, new ManagerListener() {
   public Executor getExecutor() {
       return null;
   }

   public void receiveConfigInfo(String configInfo) {
      // 客户端处理数据的逻辑

   }
});
```

参数的说明：

group和dataId为String类型，二者结合为diamond-server端保存数据的惟一key

ManagerListener 是客户端注册的数据监听器， 它的作用是在运行中接受变化的配置数据，然后回调receiveConfigInfo\(\)方法，执行客户端处理数据的逻辑。如果要在运行中对变化的配置数据进行处理，就一定要注册ManagerListener

### 1.4.3.获取配置数据

```
String configInfo = manager.getAvailableConfigInfomation(timeout);
```

diamond-server端保存的配置全都为文本类型，返回给客户端的配置数据为java.lang.String类型，timeout为从网络获取配置数据的超时时间。客户端调用每次调用该方法，都能够保证获取一份最新的可用的配置数据。

## 1.5.核心原理

diamond核心原理主要包括server集群的数据同步、client获取server地址、client从server获取数据、client运行时感知server的数据变化，这四部分。


### 1.5.1. server集群的数据同步

diamond-server将数据存储在mysql和本地文件中，mysql是一个中心，diamond认为存储在mysql中的数据绝对正确，除此之外，server会将数据存储在本地文件中。


**同步数据有两种方式：**
server写数据时，先将数据写入mysql，然后写入本地文件，写入完成后发送一个HTTP请求给集群中的其他server，其他server收到请求，从mysql中dump刚刚写入的数据至本地文件。

server启动后会启动一个定时任务，定时从mysql中dump所有数据至本地文件。

### 1.5.2.client获取server地址

diamond-client在使用时没有指定server地址的代码，地址获取对用户是透明的。server地址存储在一台具有域名的机器上的HTTP server中，我们称它为地址服务器，diamond-client使用前需要在本地进行正确的域名绑定，启动时它会根据域名绑定，去对应环境的地址服务器上获取diamond-server地址列表。获取的地址列表，会保存在client本地，当出现网络异常，无法从网络获取地址列表时，client会使用本地保存的地址列表。client启动后会启动一个定时任务，定时从HTTP server上获取地址列表并保存在本地，以保证地址是最新的。

### 1.5.3. client从server获取数据
client调用getAvailableConfigInfomation\(\)， 即可获取一份最新的可用的配置数据，获取过程实际上是拼接http url，使用http-client调用http method的过程。为了避免短时间内大量的获取数据请求发向server，client端实现了一个带有过期时间的缓存，client将本次获取到的数据保存在缓存中，在过期时间内的所有请求，都返回缓存内的数据，不向server发出请求。

### 1.5.4. client运行时感知server的数据变化
这是diamond最为核心的一个功能。这个特性是通过比较client和server的数据的MD5值实现的。server在启动时，会将所有数据的MD5加载到内存中（MD5根据某算法得出，保证数据内容不同，MD5不同，MD5存储在mysql中），数据更新时，会更新内存中对应的MD5。client在启动并第一次获取数据后，会将数据的MD5保存在内存中，并且在启动时会启动一个定时任务，定时去server检查数据是否变化。每次检查时，client将MD5传给server，server比较传来的MD5和自身内存中的MD5是否相同，如果相同，说明数据没变，返回一个标示数据不变的字符串给client；如果不同，说明数据变了，返回变化数据的dataId和group给client.  client收到变化数据的dataId和group，再去server请求一次数据，拿回数据后回调监听器。

## 1.6.diamond架构
**diamond服务是一个集群，是一个去除单点的协作集群。如下图所示：**

# 2.怎么使用

# 3.总结

# 4.参考

阿里中间件diamond：

[https://blog.csdn.net/zh\_winer/article/details/50395024](https://blog.csdn.net/zh_winer/article/details/50395024)


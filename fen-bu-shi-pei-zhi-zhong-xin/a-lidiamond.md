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

1.2.5.pushit

pushit是一个轻量级的消息通知服务组件，用来为diamond做实时通知服务，通知客户端数据的变化，它也是CS的结构，服务端搭建步骤如下：

在pushit源代码根目录下，执行mvn clean package assembly:assembly -Dmaven.test.skip命令，成功后会在pushit/target目录中看到pushit-pushit.tar.gz包。

执行tar -xzvf  pushit-pushit.tar.gz，进行解压。

进入pushit目录，建立logs目录，在logs目录中建立pushit.log文件。

进入pushit/bin目录，执行./pushit-startup.sh ../conf/server.properties命令，启动pushit-server

1.2.6.redis

redis用来存放一些跟统计相关的信息。

redis的安装请参考redis的官方文档。

安装完成后，请在diamond-server的配置文件redis.properties中填写对应的信息。

完成以上6步后，server端的搭建就完成了。

## 1.3.发布数据

diamond发布数据通过手工的方式进行。

修改diamond-server的配置文件user.properties，以k=v的方式添加登录diamond-server的用户名和密码。

在浏览器中输入http://ip:port/diamond-server/，ip和port为server搭建的第（2）步中的地址和端口，登录后进入后台管理界面，然后点击“配置信息管理”—— “添加配置信息”，在输入框中输入dataId、group、内容，最后点击“提交”即可。

成功后，可以在“配置信息管理”中查询到发布的数据。

## 1.4.订阅数据

diamond客户端API主要提供了订阅数据的功能.

### 1.4.1.客户端获取服务端地址

获取服务端地址对客户端是透明的，客户端仅仅需要在本地进行如下域名绑定即可：

domain  ip

其中，domain的值与diamond-utils工程下的com.taobao.diamond.common.Constants类中的DEFAULT\_DOMAINNAME和DAILY\_DOMAINNAME的值相同，ip为server搭建第（4）步中的http server地址。

1.4.2.创建订阅者

```

```

参数的说明：

group和dataId为String类型，二者结合为diamond-server端保存数据的惟一key

ManagerListener 是客户端注册的数据监听器， 它的作用是在运行中接受变化的配置数据，然后回调receiveConfigInfo\(\)方法，执行客户端处理数据的逻辑。如果要在运行中对变化的配置数据进行处理，就一定要注册ManagerListener

（3）获取配置数据

String configInfo = manager.getAvailableConfigInfomation\(timeout\);

diamond-server端保存的配置全都为文本类型，返回给客户端的配置数据为java.lang.String类型，timeout为从网络获取配置数据的超时时间。客户端调用每次调用该方法，都能够保证获取一份最新的可用的配置数据。

# 2.怎么使用

# 3.总结

# 4.参考

阿里中间件diamond：

[https://blog.csdn.net/zh\_winer/article/details/50395024](https://blog.csdn.net/zh_winer/article/details/50395024)


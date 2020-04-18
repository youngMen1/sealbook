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

————————————————

版权声明：本文为CSDN博主「zh\_winer」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/zh\_winer/article/details/50395024

# 2.怎么使用

# 3.总结

# 4.参考

阿里中间件diamond：

[https://blog.csdn.net/zh\_winer/article/details/50395024](https://blog.csdn.net/zh_winer/article/details/50395024)


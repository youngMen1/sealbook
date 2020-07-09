# 1.Springboot启动异常总结

## 1.1.Intellij IDEA运行报Command line is too long解法

![img](/assets/import.png)

```
<property name="dynamic.classpath" value="true"/>
```

## 1.2.SpringBoot启动异常

![](/assets/springboot启动异常.png)

# 2.Mysql异常

## 2.1.MySql Host is blocked because of many connection errors 解决方法

![](/static/image/微信截图_20200706141311.png)

```
message from server: "Host '192.168.1.28' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'"
```

**原因：**  
同一个ip在短时间内产生太多（超过mysql数据库max\_connection\_errors的最大值）中断的数据库连接而导致的阻塞；  
**解决方法：**

```
1、提高允许的max_connect_errors数量（这种方法不彻底，后期还可能导致异常出现）：

进入Mysql数据库查看max_connect_errors： show variables like ‘max_connect_errors’;

修改max_connect_errors的数量为1000： set global max_connect_errors = 1000;

查看是否修改成功：show variables like 'max_connect_errors';

2、使用mysqladmin flush-hosts 命令清理一下hosts文件

whereis mysqladmin查找mysqladmin的路径

使用命令修改：

/usr/bin/mysqladmin flush-hosts -h192.168.1.121 -uroot -p

备注：

配置有master/slave主从数据库的要把主库和从库都修改一遍

第二步也可以在数据库中进行，命令如下：flush hosts;
mysql> flush hosts;

没有权限的用户先授权或者用root操作

grant all privileges on *.* to 'wang'@'%' identified by 'MyNewPass4!';
```

## 3.Springboot报错说 Failed to parse multipart servlet request; nested exception is java.io.IOException
**原因**
1.该异常是如何产生的

我是通过gentman，发送一个post请求，导致该异常的。从上面的异常信息来看，是因为该目录[/tmp/tomcat.1428942566812653608.8090/work/Tomcat/localhost/ROOT]，不存在导致的。

**解决方案**
1.重启你的项目就可以了（我采用的这种）
2.在application.yml文件中设置multipart location ，并重启项目
```
spring:
  http:
    multipart:
      location: /data/upload_tmp
```


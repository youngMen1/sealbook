## 47.107.152.93 {#4710715293}

# 一、检查Linux系统中是否已安装 MySQL

![](/assets/微信截图_20190712114852.png)

返回空值的话，就说明没有安装 MySQL 。

注意:

在新版本的CentOS7中，默认的数据库已更新为了Mariadb，而非 MySQL，所以执行 yum install mysql 命令只是更新Mariadb数据库，并不会安装 MySQL 。  
如果已安装的 MySQL 版本不是想要的版本。需要把原来的 MySQL 卸载。

```
yum remove mysql mysql-server mysql-libs mysql-common
rm -rf /var/lib/mysql
rm -f /etc/my.cnf
```

**注意:**使用yum命令卸载，因为yum命令可以自动删除与mysql相关的依赖；如果使用rpm命令卸载，则还需要手动去删除和mysql相关的文件。


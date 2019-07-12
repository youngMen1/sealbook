## 47.107.152.93 {#4710715293}

# 一、检查Linux系统中是否已安装 MySQL

```
rpm -qa | grep mysql
```

![](/assets/微信截图_20190712114852.png)

返回空值的话，就说明没有安装 MySQL 。

注意:

在新版本的CentOS7中，默认的数据库已更新为了Mariadb，而非 MySQL，所以执行 yum install mysql 命令只是更新Mariadb数据库，并不会安装 MySQL 。  
如果已安装的 MySQL 版本不是想要的版本。需要把原来的 MySQL 卸载,命令如下:

```
yum remove mysql mysql-server mysql-libs mysql-common
rm -rf /var/lib/mysql
rm -f /etc/my.cnf
```

**注意:**使用yum命令卸载，因为yum命令可以自动删除与mysql相关的依赖；如果使用rpm命令卸载，则还需要手动去删除和mysql相关的文件。

# 第二步：查看已安装的 Mariadb 数据库版

```
rpm -qa | grep mariadb | xargs rpm -e --nodeps
```

![](/assets/微信截图_20190712115831.png)

# 第三步：卸载已安装的 Mariadb 数据库

```
rpm -qa | grep mariadb | xargs rpm -e --nodeps
```

![](/assets/微信截图_20190712132451.png)

# 第四步：再次查看已安装的 Mariadb 数据库版本，确认是否卸载完成

```
rpm -qa | grep -i mariadb
```

![](/assets/微信截图_20190712132632.png)

# 第五步：下载并安装mysql的yum源

```
1、选择一个目录放置下载的mysql的yum源文件
[root@itheima java]# mkdir mysql
[root@itheima java]# cd mysql/
[root@itheima mysql]#
[root@itheima mysql]# pwd
/usr/local/java/mysql

2、下载mysql的yum源
[root@itheima mysql]# wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
--2018-12-28 18:23:22--  http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
正在解析主机 repo.mysql.com (repo.mysql.com)... 23.41.23.231
正在连接 repo.mysql.com (repo.mysql.com)|23.41.23.231|:80... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：6140 (6.0K) [application/x-redhat-package-manager]
正在保存至: “mysql-community-release-el7-5.noarch.rpm”

100%[=========================================================================>] 6,140       --.-K/s 用时 0s

2018-12-28 18:23:23 (750 MB/s) - 已保存 “mysql-community-release-el7-5.noarch.rpm” [6140/6140])

3、安装mysql的yum源
[root@itheima mysql]# rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

如下图所示：

安装完成之后，会在 /etc/yum.repos.d/ 目录下新增 mysql-community.repo 、mysql-community-source.repo 两个 yum 源文件。

执行 yum repolist all \| grep mysql 命令查看可用的 mysql 安装文件。

# 第六步：正式安装mysql，需要使用yum命令安装。在安装mysql之前需要安装mysql的下载源。需要从oracle的官方网站下载。上面我们已经安装好了！

```
yum install mysql-community-server
```

![](/assets/微信截图_20190712133354.png)

# 第七步：看安装过程检查mysql是否安装成功

# 第八步：启动mysql。

```
service mysqld start
或者如下命令也可以
systemctl start mysqld.service      #启动 mysql
systemctl restart mysqld.service    #重启 mysql
systemctl stop mysqld.service       #停止 mysql
systemctl enable mysqld.service     #设置 mysql 开机启动
```

# 第九步：需要给root用户设置密码。有两种方式：

**方式一**：mysql5.6 安装完成后，它的 root 用户的密码默认是空的，我们需要及时用 mysql 的 root 用户登录（**第一次直接回车，不用输入密码**），并修改密码。

```
# mysql -u root
mysql> show database;
mysql> use mysql;
mysql> update user set password=PASSWORD("这里输入root用户密码") where User='root';
mysql> quit;
```

```
方式二：/usr/bin/mysqladmin -u root password 'new-password'　　#为root账号设置密码
```

# 第十步：使用root账号登录mysql。

```
mysql -u root -p xxxx
```

# 第十一步：需要先登录到mysql，设置远程连接授权，执行以下命令，为root 用户添加远程登录的能力。

```
mysql>use mysql
mysql>grant all privileges on *.* to 'root'@'%' identified by 'xxxxx(密码)';
mysql>flush privileges;
```

# 注意:

```
开放3306端口
```

```
由于centos7使用firewalld而不是iptables,所以：

```

```

#开放3306
firewall-cmd --permanent --add-port=3306/tcp 

systemctl restart firewalld.service

#查看端口是否开放
firewall-cmd --query-port=3306/tcp

#list
firewall-cmd --list-all
```




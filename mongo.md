## 安装mongodb

### 配置yum管理包

1.在路径/etc/yum.repos.d/下创建文件mongodb-org-3.4.repo

```
cd /etc/yum.repos.d/
touch mongodb-org-3.4.repo
```

2.在文件mongodb-org-3.4.repo中写入如下内容

```
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=
https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/

gpgcheck=1
enable=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```

### _安装mongodb（会安装mongodb-org包及其依赖包mongodb-org-server、mongodb-org-mongos、mongodb-org-shell、mongodb-org-tools_）

_　　数据库实例默认在/var/lib/mongo路径下，日志默认在/var/log/mongodb路径下，也可以通过修改/etc/mongod.conf文件的storage.dbPath和systemLog.path配置_

```
yum install -y mongodb-org
```

_3._开启mongodb服务

```
service mongod start
```

4、通过查看日志方式，验证服务开启成功，其中有一行为：\[thread1\] waiting for connections on port 27017

```
cat /var/log/mongodb/mongod.log
```

5、设置开机启动

```
chkconfig mongod on
```

6、停止mongodb服务

```
service mongod stop
```

7、重启mongodb服务

```
service mongod restart
```

### 卸载

1、停止服务

```
service mongod stop
```

2、删除安装的包

```
yum erase $(rpm -qa | grep mongodb-org)
```

3、删除数据及日志

```
rm -r /var/log/mongodb
rm -r /var/lib/mongo
```

### 参考官网：

[https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)


# 1.MySQL数据库优化技巧

## 1.1.MySQL优化三大方向
1.优化MySQL所在服务器内核(此优化一般由运维人员完成)。
2.对MySQL配置参数进行优化（my.cnf）此优化需要进行压力测试来进行参数调整。
3.对SQL语句以及表优化。

## 1.2.MySQL参数优化
1:MySQL 默认的最大连接数为 100，可以在 mysql 客户端使用以下命令查看


```
mysql> show variables like 'max_connections';
```


2:查看当前访问Mysql的线程


```
mysql> show processlist;
```


3:设置最大连接数
最大可设置16384,超过没用

```
mysql>set globle max_connections = 5000;
```



4:查看当前被使用的connections


```
mysql>show globle status like 'max_user_connections'
```






## 1.3.


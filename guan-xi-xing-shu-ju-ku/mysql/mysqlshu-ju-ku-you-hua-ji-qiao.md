# 1.MySQL数据库优化技巧

## 1.1.MySQL优化三大方向
1.优化MySQL所在服务器内核(此优化一般由运维人员完成)。
2.对MySQL配置参数进行优化（my.cnf）此优化需要进行压力测试来进行参数调整。
3.对SQL语句以及表优化。

## 1.2.MySQL参数优化
1.MySQL 默认的最大连接数为 100，可以在 mysql 客户端使用以下命令查看


```
mysql> show variables like 'max_connections';
```


2.查看当前访问Mysql的线程


```
mysql> show processlist;
```


3.设置最大连接数
最大可设置16384,超过没用

```
mysql>set globle max_connections = 5000;
```



4.查看当前被使用的connections


```
mysql>show globle status like 'max_user_connections'
```

## 1.3.MySQL语句性能优化的16条经验
① 为查询缓存优化查询
② EXPLAIN 我们的SELECT查询(可以查看执行的行数)
③ 当只要一行数据时使用LIMIT 1
④ 为搜索字段建立索引
⑤ 在Join表的时候使用相当类型的列，并将其索引
⑥ 千万不要 ORDER BY RAND ()
⑦ 避免SELECT *
⑧ 永远为每张表设置一个ID
⑨ 可以使用ENUM 而不要VARCHAR
⑩ 尽可能的使用NOT NULL
⑪ 固定长度的表会更快
⑫ 垂直分割
⑬ 拆分打的DELETE或INSERT语句
⑭ 越小的列会越快
⑮ 选择正确的存储引擎
⑯ 小心 "永久链接"

### 1.3.1.为查询缓存优化查询

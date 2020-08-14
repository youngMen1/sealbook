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
大多数的MySQL服务器都开启了查询缓存。这是提高性能最有效的方法之一，而且这是被MySQL引擎处理的。当有很多相同的查询被执行了多次的时候，这些查询结果会被放入一个缓存中，这样后续的相同查询就不用操作而直接访问缓存结果了。
这里最主要的问题是，对于我们程序员来说，这个事情是很容易被忽略的。因为我们某些查询语句会让MySQL不使用缓存，示例如下：


```
1：SELECT username FROM user WHERE signup_date >= CURDATE()
2：SELECT username FROM user WHERE signup_date >= '2014-06-24‘
```


上面两条SQL语句的差别就是 CURDATE() ，MySQL的查询缓存对这个函数不起作用。所以，像 NOW() 和 RAND() 或是其它的诸如此类的SQL函数都不会开启查询缓存，因为这些函数的返回是会不定的易变的。所以，你所需要的就是用一个变量来代替MySQL的函数，从而开启缓存。

### 1.2.2.使用EXPLAIN关键字检测查询
使用EXPLAIN关键字可以使我们知道MySQL是如何处理SQL语句的，这样可以帮助我们分析我们的查询语句或是表结构的性能瓶颈；EXPLAIN的查询结果还会告诉我们索引主键是如何被利用的，数据表是如何被被搜索或排序的....等等。语法格式是：EXPLAIN +SELECT语句;
![](/static/image/v2-43bed9558eec0873801d2f377651e1a6_720w.png)
我们可以看到，前一个结果显示搜索了 7883 行，而后一个只是搜索了两个表的 9 和 16 行。查看rows列可以让我们找到潜在的性能问题。 

### 1.2.3.当只要一行数据时使用LIMIT 1
加上LIMIT 1可以增加性能。MySQL数据库引擎会在查找到一条数据后停止搜索，而不是继续往后查询下一条符合条件的数据记录。


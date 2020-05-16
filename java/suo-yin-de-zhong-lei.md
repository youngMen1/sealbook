# 1.基本介绍

## 1.1.索引类型

Mysql目前主要有以下几种索引类型：FULLTEXT，HASH，BTREE，RTREE。

1.FULLTEXT  
即为全文索引，目前只有MyISAM引擎支持。其可以在CREATE TABLE ，ALTER TABLE ，CREATE INDEX 使用，不过目前只有 CHAR、VARCHAR ，TEXT 列上可以创建全文索引。

全文索引并不是和MyISAM一起诞生的，它的出现是为了解决WHERE name LIKE “%word%"这类针对文本的模糊查询效率较低的问题。

2.HASH  
由于HASH的唯一（几乎100%的唯一）及类似键值对的形式，很适合作为索引。

HASH索引可以一次定位，不需要像树形索引那样逐层查找,因此具有极高的效率。但是，这种高效是有条件的，即只在“=”和“in”条件下高效，对于范围查询、排序及组合索引仍然效率不高。

3.BTREE  
BTREE索引就是一种将索引值按一定的算法，存入一个树形的数据结构中（二叉树），每次查询都是从树的入口root开始，依次遍历node，获取leaf。这是**MySQL里默认和最常用的索引类型。**

4.RTREE  
RTREE在MySQL很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种。

## 1.2.索引种类

* 普通索引\(INDEX\)：仅加速查询

* 唯一索引\(UNIQUE\) ：加速查询 + 列值唯一（可以有null）

* 主键索引\(PRIMARY KEY\)：加速查询 + 列值唯一（不可以有null）+ 表中只有一个

* 组合索引：多列值组成一个索引，专门用于组合搜索，其效率大于索引合并

* 全文索引\(FULLTEXT\)：对文本的内容进行分词，进行搜索

**注意**

索引合并，使用多个单列索引组合搜索

覆盖索引，select的数据列只用从索引中就能够取得，不必读取数据行，换句话说查询列要被所建的索引覆盖

# 1.3.Mysql索引类型Normal,Unique,Full Text区别及索引方法Btree,Hash的区别

##### 索引方法（索引内部使用的算法）

### Normal: {#normal}

表示普通索引,没有任何限制,仅加速查询

### Unique: {#unique}

表示唯一索引，不允许重复的索引，如果该字段信息保证不会重复。例如身份证号用作索引时，可设置为unique。

### Full Text: {#unique}

表示全文搜索的索引，仅可用于 MyISAM 表。 FULLTEXT 用于搜索很长一篇文章的时候，效果最好。用在比较短的文本，切记对于大容量的数据表，生成全文索引是一个非常消耗时间非常消耗硬盘空间的做法。

## btree索引和hash索引的区别 {#btree索引和hash索引的区别}

**HASH**

**Hash**这个词，可以说，自打我们开始码的那一天起，就开始不停地见到和使用到了。其实，hash就是一种（key=&gt;value）形式的键值对，如数学中的函数映射，允许多个key对应相同的value，但不允许一个key对应多个value。正是由于这个特性，hash很适合做索引，为某一列或几列建立hash索引，就会利用这一列或几列的值通过一定的算法计算出一个hash值，对应一行或几行数据（这里在概念上和函数映射有区别，不要混淆）。在java语言中，每个类都有自己的hashcode\(\)方法，没有显示定义的都继承自object类，该方法使得每一个对象都是唯一的，在进行对象间equal比较，和序列化传输中起到了很重要的作用。hash的生成方法有很多种，足可以保证hash码的唯一性，例如在MongoDB中，每一个document都有系统为其生成的唯一的objectID（包含时间戳，主机散列值，进程PID，和自增ID）也是一种hash的表现。额，我好像扯远了-\_-!

由于hash索引可以一次定位，不需要像树形索引那样逐层查找,因此具有极高的效率。那为什么还需要其他的树形索引呢？

在这里愚安就不自己总结了。引用下园子里其他大神的文章：来自 14的路 的[MySQL的btree索引和hash索引的区别](http://www.cnblogs.com/vicenteforever/articles/1789613.html)

（1）Hash 索引仅仅能满足"=","IN"和"&lt;=&gt;"查询，不能使用范围查询。  
由于 Hash 索引比较的是进行 Hash 运算之后的 Hash 值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的 Hash 算法处理之后的 Hash 值的大小关系，并不能保证和Hash运算前完全一样。  
（2）Hash 索引无法被用来避免数据的排序操作。  
由于 Hash 索引中存放的是经过 Hash 计算之后的 Hash 值，而且Hash值的大小关系并不一定和 Hash 运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算；  
（3）Hash 索引不能利用部分索引键查询。  
对于组合索引，Hash 索引在计算 Hash 值的时候是组合索引键合并后再一起计算 Hash 值，而不是单独计算 Hash 值，所以通过组合索引的前面一个或几个索引键进行查询的时候，Hash 索引也无法被利用。  
（4）Hash 索引在任何时候都不能避免表扫描。  
前面已经知道，Hash 索引是将索引键通过 Hash 运算之后，将 Hash运算结果的 Hash 值和所对应的行指针信息存放于一个 Hash 表中，由于不同索引键存在相同 Hash 值，所以即使取满足某个 Hash 键值的数据的记录条数，也无法从 Hash 索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果。  
（5）Hash 索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高。  
对于选择性比较低的索引键，如果创建 Hash 索引，那么将会存在大量记录指针信息存于同一个 Hash 值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据的访问，而造成整体性能低下。

## 1.4.索引的创建方式

索引的创建方式

\(1\)主键索引的创建方式：

方式1：ALTER TABLE \`table\_name\` ADD PRIMARY KEY \( \`column\` \)

比如：ALTER TABLE users ADD PRIMARY KEY \( id \)

方式2：创建表的时候指定主键

\(2\)唯一索引的创建方式

方式1：ALTER TABLE \`table\_name\` ADD UNIQUE  \[indexName\] \(\`column\`\)

比如：ALTER TABLE users ADD UNIQUE \( id \)

方式2：CREATE UNIQUE INDEX index\_name ON table\_name \(column\_name\)

比如：CREATE UNIQUE INDEX index\_users ON users\(id\)

\(3\)普通索引的创建方式

方式1：ALTER TABLE \`table\_name\` ADD INDEX index\_name \( \`column\` \)

比如：ALTER TABLE users ADD INDEX index\_users\( id \)

方式2：CREATE INDEX index\_name ON table\_name \(column\_name\)

比如：CREATE INDEX index\_users ON users \(column\_name\)

\(4\)全文索引的创建方式

方式1：ALTER TABLE \`table\_name\` ADD FULLTEXT \( \`column\` \)

比如：ALTER TABLE users ADD FULLTEXT \( id \)


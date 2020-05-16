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

Hash 索引结构的特殊性，其检索效率非常高，索引的检索可以一次定位，不像B-Tree 索引需要从根节点到枝节点，最后才能访问到页节点这样多次的IO访问，所以 Hash 索引的查询效率要远高于 B-Tree 索引。

可 能很多人又有疑问了，既然 Hash 索引的效率要比 B-Tree 高很多，为什么大家不都用 Hash 索引而还要使用 B-Tree 索引呢？任何事物都是有两面性的，Hash 索引也一样，虽然 Hash 索引效率高，但是 Hash 索引本身由于其特殊性也带来了很多限制和弊端，主要有以下这些。

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

**BTREE**

BTREE索引就是一种将索引值按一定的算法，存入一个树形的数据结构中，相信学过数据结构的童鞋都对当初学习二叉树这种数据结构的经历记忆犹新，反正愚安我当时为了软考可是被这玩意儿好好地折腾了一番，不过那次考试好像没怎么考这个。如二叉树一样，每次查询都是从树的入口root开始，依次遍历node，获取leaf。

BTREE在MyISAM里的形式和Innodb稍有不同

在 Innodb里，有两种形态：一是primary key形态，其leaf node里存放的是数据，而且不仅存放了索引键的数据，还存放了其他字段的数据。二是secondary index，其leaf node和普通的BTREE差不多，只是还存放了指向主键的信息.

而在MyISAM里，主键和其他的并没有太大区别。不过和Innodb不太一样的地方是在MyISAM里，leaf node里存放的不是主键的信息，而是指向数据文件里的对应数据行的信息.

**RTREE**

RTREE在mysql很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种。

相对于BTREE，RTREE的优势在于范围查找.

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


## NoSQL 数据库分类 {#10}

| 类型 | 部分代表 | 特点 |
| :---: | :---: | :---: |
| 列存储 | Hbase/Cassandra/Hypertable | 顾名思议，是按列存储数据的，最大的特点是方便存储结构化和半结构化数据，方便做数据压缩，对针对某一列或者某几列的查询有非常大的IO优势。 |
| 文档存储 | MongoDB/CouchDB | 文档存储一般用类似json的格式存储，存储的内容是文档型的，这样也就有机会对某些字段建立索引，实现关系数据库的某些功能。 |
| key-value存储 | Tokyo Cabinet/Tyrant Berkeley DB/MemcacheDB/Redis | 可以通过key快速查询到其value。一般来说，存储不管value的格式，照单全收。\(redis包含了其他功能\) |
| 图存储 | Neo4j/FlockDB | 图形关系的最佳存储，使用传统关系数据库来解决的话性能低下，而且设计使用不方便。 |
| 对象存储 | db4o/Versant |  |
| xml数据库 | BerKeley DB XML/BaseX |  |

参考:

[https://yq.aliyun.com/articles/523620?spm=5176.10695662.1996646101.searchclickresult.327d3527aTDmrc](https://yq.aliyun.com/articles/523620?spm=5176.10695662.1996646101.searchclickresult.327d3527aTDmrc)


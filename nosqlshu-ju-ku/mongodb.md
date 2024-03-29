## 1.**MongoDB 数据类型**![](/assets/mongodb数据类型.png)

```
{
        "desc" : "conn632530",
        "threadId" : "140298196924160",
        "connectionId" : 632530,
        "client" : "11.192.159.236:57052",
        "active" : true,
        "opid" : 1008837885,
        "secs_running" : 0,
        "microsecs_running" : NumberLong(70),
        "op" : "update",
        "ns" : "mygame.players",
        "query" : {
            "uid" : NumberLong(31577677)
        },
        "numYields" : 0,
        "locks" : {
            "Global" : "w",
            "Database" : "w",
            "Collection" : "w"
        },
        ....
    },
```

## 特性

MongoDB 的主要特性，可以对照自己的业务需求看看，匹配的越多，用 MongoDB 就越合适。

| MongoDB 特性 | 优势 |
| :--- | :--- |
| 事务支持 | MongoDB 目前只支持单文档事务，需要复杂事务支持的场景暂时不适合 |
| 灵活的文档模型 | JSON 格式存储最接近真实对象模型，对开发者友好，方便快速开发迭代 |
| 高可用复制集 | 满足数据高可靠、服务高可用的需求，运维简单，故障自动切换 |
| 可扩展分片集群 | 海量数据存储，服务能力水平扩展 |
| 高性能 | mmapv1、wiredtiger、mongorocks（rocksdb）、in-memory 等多引擎支持满足各种场景需求 |
| 强大的索引支持 | 地理位置索引可用于构建 各种 O2O 应用、文本索引解决搜索的需求、TTL索引解决历史数据自动过期的需求 |
| Gridfs | 解决文件存储的需求 |
| aggregation & mapreduce | 解决数据分析场景需求，用户可以自己写查询语句或脚本，将请求都分发到 MongoDB 上完成 |
| 分布式、schema | Mongo“天生骄傲”，从设计之初就考虑了分布式，而MySQL则折腾比较多，MongoDB是非关系型数据库，里面基本不存在schema\(模式\)；而关系型数据库MySQL在这一方面则非常强。 |

比如游戏、物流、电商、内容管理、社交、物联网、视频直播等，以下是几个实际的应用案例。

* 游戏场景，使用 MongoDB 存储游戏用户信息，用户的装备、积分等直接以内嵌文档的形式存储，方便查询、更新
* 物流场景，使用 MongoDB 存储订单信息，订单状态在运送过程中会不断更新，以 MongoDB 内嵌数组的形式来存储，一次查询就能将订单所有的变更读取出来。
* 社交场景，使用 MongoDB 存储存储用户信息，以及用户发表的朋友圈信息，通过地理位置索引实现附近的人、地点等功能
* 物联网场景，使用 MongoDB 存储所有接入的智能设备信息，以及设备汇报的日志信息，并对这些信息进行多维度的分析
* 视频直播，使用 MongoDB 存储用户信息、礼物信息等

| 应用特征 | Yes / No |
| :--- | :--- |
| 应用不需要事务及复杂 join 支持 | 必须 Yes |
| 新应用，需求会变，数据模型无法确定，想快速迭代开发 | ？ |
| 应用需要2000-3000以上的读写QPS（更高也可以） | ？ |
| 应用需要TB甚至 PB 级别数据存储 | ? |
| 应用发展迅速，需要能快速水平扩展 | ? |
| 应用要求存储的数据不丢失 | ? |
| 应用需要99.999%高可用 | ? |
| 应用需要大量的地理位置查询、文本查询 | ？ |

如果上述有1个 Yes，可以考虑 MongoDB，2个及以上的 Yes，选择MongoDB绝不会后悔。

## ![](/assets/微信截图_20190727140406.png)

---

## 2.参考:

[https://yq.aliyun.com/articles/73884?spm=5176.10695662.1996646101.searchclickresult.12535a51yAjKGr](https://yq.aliyun.com/articles/73884?spm=5176.10695662.1996646101.searchclickresult.12535a51yAjKGr)

[https://yq.aliyun.com/articles/702430?spm=5176.10695662.1996646101.searchclickresult.4a5a6c5fRTpOQV](https://yq.aliyun.com/articles/702430?spm=5176.10695662.1996646101.searchclickresult.4a5a6c5fRTpOQV)

排查MongoDB CPU使用率高的问题

[https://help.aliyun.com/document\_detail/62224.html?spm=5176.10695662.1996646101.searchclickresult.11f018c1AXuDIq](https://help.aliyun.com/document_detail/62224.html?spm=5176.10695662.1996646101.searchclickresult.11f018c1AXuDIq)


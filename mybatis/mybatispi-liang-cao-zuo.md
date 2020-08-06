# 1.Mybatis批量操作
## 1.1.批量添加
批量添加在做大量数据插入到mysql时,效率相对单条遍历插入大大提高;
但是数据是基于数据库层面做的约束的话,在插入的数据中有一个数据有误,整个批量操作全部回滚;
适用场景:数据迁移时使用


```
    <insert id="batchInsertMember" parameterType="java.util.List">
        insert into crm_member(
        id,
        name,
        type,
        phone,
        link,
        )
        values
        <foreach collection="list" item="item" index="index" separator=",">
            (
            #{item.id},
            #{item.name},
            #{item.type},
            #{item.phone},
            #{item.link}
            )
        </foreach>
    </insert>
```
**生成对应的sql**


```
insert into 
 　　crm_member3 　　(
　　　　id, 
　　　　name, 
　　　　type, 
　　　　phone,
　　　　link
　　　　)
 values
 　　(?,?,?,?,?),
 　　(?,?,?,?,?),
 　　(?,?,?,?,?),
 　　(?,?,?,?,?)
```





## 1.2.批量更新



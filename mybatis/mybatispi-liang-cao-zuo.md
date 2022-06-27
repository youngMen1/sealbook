# 1.Mybatis批量操作

## 1.1.批量添加

批量添加在做大量数据插入到mysql时,效率相对单条遍历插入大大提高;  
但是数据是基于数据库层面做的约束的话,在插入的数据中有一个数据有误,整个批量操作全部回滚;  
适用场景:数据迁移时使用

```
<sql id="Base_Column_List">
  id, name, type, phone, link
</sql>


<insert id="batchInsertMember" parameterType="java.util.List">
    insert into crm_member(
    /*方法一*/
    id,
    name,
    type,
    phone,
    link
)
    /*方法二*/
    (<include refid="Base_Column_List"/>)

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

### 1.2.1.单个字段写法一

Mybatis写法如下：

```
<update id="updateBatch"  parameterType="java.util.List">  
    <foreach collection="list" item="item" index="index" open="" close="" separator=";">
        update course
        <set>
            name=#{item.name}
        </set>
        where id = #{item.id}
    </foreach>      
</update>


<update id="updateBatchByIds" parameterType="java.util.List">
        update task_user_relation set is_del = 1
        where id in
        <foreach collection="taskUserRelationIds" item="id" open="(" separator="," close=")">
            #{id}
        </foreach>
    </update>
```

一条记录update一次，性能比较差，容易造成阻塞。

MySQL没有提供直接的方法来实现批量更新，但可以使用case when语法来实现这个功能。

```
UPDATE course
    SET name = CASE id 
        WHEN 1 THEN 'name1'
        WHEN 2 THEN 'name2'
        WHEN 3 THEN 'name3'
    END, 
    title = CASE id 
        WHEN 1 THEN 'New Title 1'
        WHEN 2 THEN 'New Title 2'
        WHEN 3 THEN 'New Title 3'
    END
WHERE id IN (1,2,3)
```

### 1.2.2.多个字段写法二

为了提升操作数据的效率,第一想到的是做批量操作,直接上批量更新代码:

```
<update id="updateBatch" parameterType="list">
            update course
            <trim prefix="set" suffixOverrides=",">
             <trim prefix="peopleId =case" suffix="end,">
                 <foreach collection="list" item="i" index="index">
                         <if test="i.peopleId!=null">
                          when id=#{i.id} then #{i.peopleId}
                         </if>
                 </foreach>
              </trim>
              <trim prefix=" roadgridid =case" suffix="end,">
                 <foreach collection="list" item="i" index="index">
                         <if test="i.roadgridid!=null">
                          when id=#{i.id} then #{i.roadgridid}
                         </if>
                 </foreach>
              </trim>

              <trim prefix="type =case" suffix="end," >
                 <foreach collection="list" item="i" index="index">
                         <if test="i.type!=null">
                          when id=#{i.id} then #{i.type}
                         </if>
                 </foreach>
              </trim>
       <trim prefix="unitsid =case" suffix="end," >
                  <foreach collection="list" item="i" index="index">
                          <if test="i.unitsid!=null">
                           when id=#{i.id} then #{i.unitsid}
                          </if>
                  </foreach>
           </trim>
             </trim>
            where
            <foreach collection="list" separator="or" item="i" index="index" >
              id=#{i.id}
          </foreach>
</update>
```

**生成的sql语句**

```
update
　　crm_member
set
dept_id =case
　　when id=? then ?
　　when id=? then ?
　　when id=? then ?
　　when id=? then ?
end,
sys_user_id =case
　　when id=? then ?
　　when id=? then ?
　　when id=? then ?
　　when id=? then ?
end,
public_area_id =case
　　when id=? then ?
　　when id=? then ?
　　when id=? then ?
　　when id=? then ?
end
where
　　id=?
　　or id=?
　　 or id=?
　　or id=?
　　 or id=?
```

## 1.3.批量删除

### 1.3.1.如果传入的是单参数且参数类型是一个List的时候，collection属性值为list

```
<delete id="deleteByLogic"  parameterType = "java.util.List">
     delete from user where 1>2
         or id in
      <foreach collection="list"  item="item" open="(" separator="," close=")"  >
           #{item}
      </foreach>
</delete>
```

### 1.3.2.如果传入的是单参数且参数类型是一个array数组的时候， 参数类型为parameterType="int"     集合    collection的属性值为array

```
<delete id="deleteByLogic"  parameterType = "java.util.List">
     delete from user where 1>2
         or id in
     <foreach item="item" collection="array" open="(" separator="," close=")">
            #{item}
     </foreach>
</delete>
```

## 1.4.批量查询

# 2.参考

mybatis 实现批量更新：[https://www.cnblogs.com/joeblackzqq/p/10892699.html](https://www.cnblogs.com/joeblackzqq/p/10892699.html)  
Mybatis 批量添加,批量更新: [https://www.cnblogs.com/djq-jone/p/10754742.html](https://www.cnblogs.com/djq-jone/p/10754742.html)  
mybatis批量插入、批量更新和批量删除：[https://www.jianshu.com/p/041bec8ae6d3](https://www.jianshu.com/p/041bec8ae6d3)


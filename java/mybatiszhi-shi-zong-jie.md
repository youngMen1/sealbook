# 1.Mybatis知识总结

## 1.1.Mybatis中`<![CDATA[]]>`的作用

在使用Mybatis 时我们sql是写在xml 映射文件中，如果写的sql中有一些特殊的字符的话，在解析xml文件的时候会被转义，但我们不希望他被转义，所以我们要使用`<![CDATA[ ]]>`来解决。

```
<![CDATA[   ]]> 是什么，这是XML语法。在CDATA内部的所有内容都会被解析器忽略。

如果文本包含了很多的"<"字符 <=和"&"字符——就象程序代码一样，那么最好把他们都放到CDATA部件中。

但是有个问题那就是
 <if test="">   
 </if>   
 <where>   
 </where>  
 <choose>  
 </choose>  
 <trim>  
 </trim> 
 等这些标签都不会被解析，所以我们只把有特殊字符的语句放在 
 <![CDATA[   ]]>  
 尽量缩小 <![CDATA[  ]]> 的范围。
```


## 1.2.mybatis传入混合参数（多个不同类型的参数）

#### 调用的接口
   `List<SpecialOrderListVO> listSpecialOrder(@Param("id") String id, @Param("specialOrderVO") SpecialOrderVO specialOrderVO);`

#### Mapper.xml文件

```
 <select id="listSpecialOrder" resultMap="ResultSpecialOrderListVOMap">
        SELECT
        o.order_no,
        o.member_name,
        o.member_phone,
        o.order_amount,
        o.staff_name,
        o.goods_type,
        o.create_time,
        o.pay_mode,
        o.pay_time,
        o.status,
        c.name
        FROM
        special_orders so
        LEFT JOIN orders o ON o.order_no = so.order_no
        LEFT JOIN club c ON c.id = o.club_id
        WHERE 1 = 1
        <if test="id != null and id != ''">
        AND so.special_business_approval_id = #{id}
        </if>
        <if test="specialOrderVO.orderNo != null and specialOrderVO.orderNo != ''">
            AND o.order_no = #{specialOrderVO.orderNo}
        </if>
        <if test="specialOrderVO.memberName != null and specialOrderVO.memberName != ''">
            AND o.member_name = #{specialOrderVO.memberName}
        </if>
        <if test="specialOrderVO.memberPhone != null and specialOrderVO.memberPhone != ''">
            AND o.member_phone = #{specialOrderVO.memberPhone}
        </if>
        <if test="specialOrderVO.clubId != null and specialOrderVO.clubId != ''">
            AND o.club_id = #{clubId}
        </if>
        <if test="specialOrderVO.status != null and specialOrderVO.status != ''">
            AND o.status = #{specialOrderVO.status}
        </if>
        <if test="specialOrderVO.goodsType != null and specialOrderVO.goodsType != ''">
            AND o.goods_type = #{specialOrderVO.goodsType}
        </if>
         ORDER BY so.create_time DESC
    </select>

```



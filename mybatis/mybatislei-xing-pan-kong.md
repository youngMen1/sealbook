# 1.Mybatis类型判空

## 1.1.int/Integer类型的参数为空判断

```
<if test="orderStatus != null">
    order_status= #{orderStatus},
</if>
```

## 1.2.String类型的参数为空判断

```
<if test="userName != null and userName != ''">
   user_name = #{userName}
</if>
```

## 1.3.集合类型判空

```
    <select id="listSpecialCardOrder" resultMap="ResultSpecialOrderListVOMap">
        SELECT
            o.id,
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
            c.name,
            os.goods_name,
            m.phone
        FROM
            orders o
        LEFT JOIN order_goods os ON o.order_no = os.order_no
        LEFT JOIN club c ON c.id = o.club_id
        LEFT JOIN member_info m ON m.uid = o.staff_id
        WHERE 1 = 1
        AND (o.status = 1 OR o.status = 2)
        AND o.goods_type = 1
        <if test="orderNoList!=null and orderNoList.size()>0">
        AND o.order_no in
        <foreach collection="orderNoList" item="orders" open="(" close=")" separator=",">
            #{orders}
        </foreach>
        </if>
        <if test="memberIdList!=null and memberIdList.size()>0">
        AND o.member_id in
        <foreach collection="memberIdList" item="memberIds" open="(" close=")" separator=",">
            #{memberIds}
        </foreach>
        </if>
    </select>
```




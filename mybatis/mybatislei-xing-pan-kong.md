# 1.Mybatis类型判空

## int/Integer类型的参数为空判断

```
<if test="orderStatus!= null and orderStatus != ''">
    order_status= #{orderStatus},
</if>
```

## String类型的参数为空判断

```
<if test="userName != null and userName != ''">
   user_name = #{userName}
</if>
```




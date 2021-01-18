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

### 1.2.1.方式一：

#### 调用的接口

`public List<User> selectUserInIDs(List<Integer> ids,String name);`

#### Mapper.xml文件

```
<select id="selectUserInIDs" resultType="User">  
        select * from user where id in   
        <foreach collection="param1" item="item" open="(" separator="," close=")">  
            #{item}  
        </foreach>  
        and name = #{param2}  
    </select>
```

### 1.2.2.方式二：

#### 调用的接口

`List<SpecialOrderListVO> listSpecialOrder(@Param("id") String id, @Param("specialOrderVO") SpecialOrderVO specialOrderVO);`

#### Mapper.xml文件

```
     <resultMap id="ResultSpecialOrderListVOMap" type="com.gdfl.order.vo.SpecialOrderListVO">
        <result column="order_no" property="orderNo"/>
        <result column="member_name" property="memberName"/>
        <result column="member_phone" property="memberPhone"/>
        <result column="order_amount" property="orderAmount"/>
        <result column="staff_name" property="staffName"/>
        <result column="goods_type" property="goodsType"/>
        <result column="create_time" property="createTime"/>
        <result column="pay_mode" property="payMode"/>
        <result column="pay_time" property="payTime"/>
        <result column="status" property="status"/>
        <result column="name" property="clubName"/>
        <result column="phone" property="salePhone"/>
    </resultMap>

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

## 1.3.MySQL DATE\_FORMAT\(\) 函数

### 实例

下面的脚本使用 DATE\_FORMAT\(\) 函数来显示不同的格式。我们使用 NOW\(\) 来获得当前的日期/时间：  
DATE\_FORMAT\(NOW\(\),'%b %d %Y %h:%i %p'\)  
DATE\_FORMAT\(NOW\(\),'%m-%d-%Y'\)  
DATE\_FORMAT\(NOW\(\),'%d %b %y'\)  
DATE\_FORMAT\(NOW\(\),'%d %b %Y %T:%f'\)  
DATE\_FORMAT\(NOW\(\),'%y-%m-%d'\)

### 结果类似：

Dec 29 2008 11:45 PM  
12-29-2008  
29 Dec 08  
29 Dec 2008 16:25:46.635  
2008-29-12

```
    SELECT
    cui.id,
    cui.employee_name,
    cui.employee_mobile,
    cui.shop_name,
    cui.shop_id,
    cui.admission_time,
    ci.channel_name
    FROM channel_consumption_record cui
    INNER JOIN channel_order co ON (cui.order_id=co.order_no)
    INNER JOIN channel_info ci on (co.channel_code=ci.id)
    WHERE 1=1
    <if test="channelInfoId != null and channelInfoId != ''">
    AND ci.id = #{channelInfoId}
    </if>
    <if test="startAdmissionTime != null">
    <![CDATA[AND DATE_FORMAT(cui.admission_time,'%y-%m-%d') >= DATE_FORMAT(#{startAdmissionTime},'%y-%m-%d')]]>
    </if>
    <if test="endAdmissionTime != null">
        <![CDATA[AND DATE_FORMAT(cui.admission_time,'%y-%m-%d') <= DATE_FORMAT(#{endAdmissionTime},'%y-%m-%d')]]>
    </if>
    <if test="employeeMobile != null and employeeMobile != ''">
    AND cui.employee_mobile = #{employeeMobile}
    </if>
    <if test="shopId != null and shopId != ''">
    AND cui.shop_id = #{shopId}
    </if>
    <if test="admissionTime != null and admissionTime != ''">
    <![CDATA[AND DATE_FORMAT(cui.admission_time,'%y-%m-%d') >= DATE_FORMAT(#{admissionTime},'%y-%m-%d')]]>
    </if>
    GROUP BY cui.id,cui.employee_name,cui.employee_mobile,cui.shop_name,cui.shop_id,cui.admission_time,ci.channel_name
    ORDER BY cui.admission_time ASC
```

## 1.4.Mybatis中like模糊查询的几种写法及注意点

### 1.4.1.第一种：使用${...}

```
<if test="memberName!=null and memberName!=''">
and member_name like '%${memberName}%'
</if>
```

注意：由于$是参数直接注入的，导致这种写法，大括号里面不能注明jdbcType，不然会报错。

弊端：可能会引起sql的注入，平时尽量避免使用${...}

### 1.4.2.第二种：使用\#{...}

```
<if test="memberName!=null and memberName!=''">
and member_name like "%"#{memberName,jdbcType=VARCHAR}"%"
</if>
```

注意：因为\#{...}解析成sql语句时候，会在变量外侧自动加单引号' '，所以这里 % 需要使用双引号" "，不能使用单引号 ' '，不然会查不到任何结果。

### 1.4.3.使用CONCAT\(\)函数连接参数形式

```
 <if test="memberName!=null and memberName!=''">
and member_name LIKE CONCAT('%',#{memberName},'%')
 </if>
```

### 1.5.Mybatis映射文件里sql标签Base\_column\_List

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.gdfl.goods.member.dao.GroupTimeDao">
    <resultMap id="BaseResultMap" type="com.gdfl.goods.model.GroupTime">
        <id column="id" property="id"/>
        <result column="group_teacher_id" property="groupTeacherId"/>
        <result column="day" property="day"/>
        <result column="start_time" property="startTime"/>
        <result column="end_time" property="endTime"/>
        <result column="create_time" property="createTime"/>
        <result column="delete_status" property="deleteStatus"/>
    </resultMap>

    <sql id="Base_Column_List">
    id, group_teacher_id, day, start_time, end_time, create_time, delete_status
  </sql>

    <select id="findGroupTimeByDay" resultMap="BaseResultMap">
        SELECT
        <include refid="Base_Column_List"/>
        FROM group_time
        WHERE 1=1
        <if test="day != null and day !=''">
            day = #{day}
        </if>
        ORDER BY create_time DESC
    </select>

</mapper>
```




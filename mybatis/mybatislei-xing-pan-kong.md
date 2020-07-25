# 1.Mybatis类型判空
## int/Integer类型的参数为空判断
<if test="staging != null and staging != ''">
    staging = #{staging},
</if>
## String类型的参数为空判断
<if test="userName != null and userName != ''">
userName = #{userName}
</if>





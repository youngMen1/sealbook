# MyBatis多层Collection集合嵌套数据返回

本示例使用策略+模板+标签。策略列表详情下显示策略模板和模板标签。是一个多层一对多的嵌套关系。

## 策略响应对象类

StrategyDetailResp

```
@Data
@ToString
public class StrategyDetailResp {
    private Integer id;

    private String name;

    private String basePath;

    private String modulePath;

    private String remark;

    private Date updateTime;

    List<TemplateListResp> templateList;
    
}
```

## 模板响应对象类

TemplateListResp

```
@Data
@ToString
public class TemplateListResp {

    private Integer id;

    private Integer userId;

    private String name;

    private String content;

    private String description;

    private Byte status;

    private Integer favoCount;

    private Integer useCount;

    private Date createTime;

    private List<TagResp> tags;
    

}
```

## 标签响应对象类

TagResp

```
@Data
@ToString
public class TagResp {
    private Integer tagId;

    private String tagName;

    private Byte tagType;
}
```

可以看到StrategyDetailResp对象有一个List集合有多个TemplateListResp对象，是一个一对多关系，TemplateListResp对象有一个List集合有多个TagResp对象，是一个一对多关系。所以这样需要使用一个多重Collection集合实现。

## mapper.xml

### 实现一

```
<resultMap id="StrategyDetailMap" type="com.codegen.dao.resq.StrategyDetailResp">
  <id column="S_ID" property="id" jdbcType="INTEGER" />
  <result column="S_NAME" property="name" jdbcType="VARCHAR" />
  <result column="S_BASE_PATH" property="basePath" jdbcType="VARCHAR" />
  <result column="S_MODULE_PATH" property="modulePath" jdbcType="VARCHAR" />
  <result column="S_REMARK" property="remark" jdbcType="VARCHAR" />
  <result column="S_UPDATE_TIME" property="updateTime" jdbcType="TIMESTAMP" />

  <collection property="templateList" ofType="com.codegen.dao.resq.TemplateListResp">
    <id column="tplId" property="id" jdbcType="INTEGER" />
    <result column="tplName" property="name" jdbcType="VARCHAR" />
    <result column="tplUserId" property="userId" jdbcType="INTEGER" />
    <result column="tplContent" property="content" jdbcType="VARCHAR" />
    <result column="tplDescription" property="description" jdbcType="VARCHAR" />
    <result column="tplStatus" property="status" jdbcType="TINYINT" />
    <result column="tplFavoCount" property="favoCount" jdbcType="INTEGER" />
    <result column="tplUseCount" property="useCount" jdbcType="INTEGER" />
    <result column="tplCreateTime" property="createTime" jdbcType="TIMESTAMP" />

    <collection property="tags" ofType="com.codegen.dao.resq.TagResp">
      <id column="tagId" property="tagId" jdbcType="INTEGER"/>
      <result column="tagName" property="tagName" jdbcType="VARCHAR"/>
      <result column="tagType" property="tagType" jdbcType="TINYINT" />
    </collection>
  </collection>
  
</resultMap>

<resultMap id="NamesMap" type="string">
    <result column="user_name" />
</resultMap>

<resultMap id="RolesMap" type="string">
    <result column="role_name" />
</resultMap>
```

### 实现二

```
<resultMap id="StrategyDetailMap" type="com.codegen.dao.resq.StrategyDetailResp">
<id column="S_ID" property="id" jdbcType="INTEGER" />
<result column="S_NAME" property="name" jdbcType="VARCHAR" />
<result column="S_BASE_PATH" property="basePath" jdbcType="VARCHAR" />
<result column="S_MODULE_PATH" property="modulePath" jdbcType="VARCHAR" />
<result column="S_REMARK" property="remark" jdbcType="VARCHAR" />
<result column="S_UPDATE_TIME" property="updateTime" jdbcType="TIMESTAMP" />

<!--resultMap 对应下面id的 templateListResp-->
<collection property="templateList" resultMap="templateListResp"/>

</resultMap>

<resultMap id="templateListResp" type="com.codegen.dao.resq.TemplateListResp">
<id column="tplId" property="id" jdbcType="INTEGER" />
<result column="tplName" property="name" jdbcType="VARCHAR" />
<result column="USER_ID" property="userId" jdbcType="INTEGER" />
<result column="CONTENT" property="content" jdbcType="VARCHAR" />
<result column="tplDescription" property="description" jdbcType="VARCHAR" />
<result column="STATUS" property="status" jdbcType="TINYINT" />
<result column="tplFavoCount" property="favoCount" jdbcType="INTEGER" />
<result column="tplUseCount" property="useCount" jdbcType="INTEGER" />
<result column="CREATE_TIME" property="createTime" jdbcType="TIMESTAMP" />


<collection property="tags" ofType="com.codegen.dao.resq.TagResp">
<id column="tagId" property="tagId" jdbcType="INTEGER"/>
<result column="tagName" property="tagName" jdbcType="VARCHAR"/>
<result column="tagType" property="tagType" jdbcType="TINYINT" />
</collection>
</resultMap>


```

type：对象上面响应对象类  
property：指的是集合属性的值  
ofType：指的是集合中元素的类型

## 最后附加上SQL查询

```
<select id="findStrategyById" resultMap="StrategyDetailMap" parameterType="com.codegen.dao.req.StrategyReq">
  SELECT
      s.ID S_ID,
      s.NAME S_NAME,
      s.BASE_PATH S_BASE_PATH,
      s.MODULE_PATH S_MODULE_PATH,
      s.REMARK S_REMARK,
      s.UPDATE_TIME S_UPDATE_TIME,
      tpl.ID tplId,
      tpl.USER_ID tplUserId,
      tpl.CONTENT tplContent,
      tpl.NAME tplName,
      tpl.STATUS tplStatus,
      tpl.DESCRIPTION tplDescription,
      tpl.FAVO_COUNT tplFavoCount,
      tpl.USE_COUNT tplUseCount,
      tpl.CREATE_TIME tplCreateTime,
      tag.ID tagId,
      tag.NAME tagName,
      tag.TYPE tagType
  FROM
      t_strategy s
      LEFT JOIN t_strategy_template st ON st.STRATEGY_ID = s.ID
      LEFT JOIN t_template tpl ON tpl.ID = st.TEMPLATE_ID
      LEFT JOIN t_template_tag tt ON tt.TEMPLATE_ID = tpl.ID
      LEFT JOIN t_tag tag ON tag.ID = tt.TAG_ID
  WHERE
      s.USER_ID = #{userId, jdbcType=INTEGER}
      AND s.ID = #{id, jdbcType=INTEGER}
      AND s.IS_ACTIVE = #{isActive, jdbcType=TINYINT}
      AND st.IS_ACTIVE = #{isActive, jdbcType=TINYINT}
      AND tpl.IS_ACTIVE = #{isActive, jdbcType=TINYINT}
      AND tag.IS_ACTIVE = #{isActive, jdbcType=TINYINT}
</select>
```

# 2.参考

MyBatis多层Collection集合嵌套数据返回:

```
https://blog.csdn.net/qq_22067469/article/details/88094722
```




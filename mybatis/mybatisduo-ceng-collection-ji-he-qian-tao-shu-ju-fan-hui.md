# 1.MyBatis多层Collection集合嵌套数据返回

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

# 2.Mybatis嵌套查询结果集中包含简单数据类型集合或者多个`List<T>`的处理

## 2.1.包含多个` List(String)`属性的情况

### 实体类


```
@Data
public class User {
    private Long id;
    private List<String> names;
    private List<String> roles;
}
```
### Mapper 层


```
public interface UserMapper {
    List<User> queryUsers();
}
```
### Mapper Sql 映射文件


```
<resultMap id="UserMap" type="User">
   <result column="id" property="id" jdbcType="BIGINT" />
    <collection property="names" resultMap="NamesMap" />
    <collection property="roles" resultMap="RolesMap" />
</resultMap>

<resultMap id="NamesMap" type="string">
    <result column="user_name" />
</resultMap>

<resultMap id="RolesMap" type="string">
    <result column="role_name" />
</resultMap>

<select id="queryUsers" resultMap="UserMap">
	SELECT au.id, an.user_name, ar.role_name
    FROM ai_user au
    LEFT JOIN ai_name an ON an.user_id = au.id
    LEFT JOIN ai_role ar ON ar.user_id = au.id
</select>
```

结果输出示例


```
{"id":1,"names":["Answer","AI","AAL"],"roles":["Admin","Manager","Coder"]}
{"id":2,"names":["Iris","Ellis","Monta"],"roles":["CustomerService"]}
```


## 2.2.包含多个` List(T) `和 `List(String)`属性的情况
### 实体类


```
@Data
public class SysUser {
    private Long id;

    private String loginName;

    private String userName;

    private String email;

    private List<Long> roleIds;

    private List<SysRole> roles;
}
```


```
@Data
public class SysRole {
    private Long id;

    private String roleName;

    private String roleDesc;
}
```

### Mapper 接口


```
public interface UserMapper {
    List<SysUser> findUsers(int status);
} 
```
### `Mapper Sql `映射文件



```
<resultMap id="SysUserMap" type="com.answer.ai.entity.SysUser">
    <id column="id" property="id" jdbcType="BIGINT" />
    <result column="login_name" property="loginName" jdbcType="VARCHAR"/>
    <result column="user_name" property="userName" jdbcType="VARCHAR"/>
    <result column="email" property="email" jdbcType="VARCHAR"/>
	
	<!-- userId 和 roleStatus 为查询 findRoleById 的入参, id 和 role_status 为查询 findUsers 的结果信息 -->
	<!-- 如果查询 findRoleById 需要 roleId 作为入参, column 写法 {userId=id,roleStatus=role_status,roleId=role_id} -->
    <collection property="roles" ofType="com.answer.ai.entity.SysRole" select="findRoleById" column="{userId=id,roleStatus=role_status}" />
    <collection property="roleIds" ofType="Long">
        <constructor>
            <arg column="role_id" />
        </constructor>
    </collection>
</resultMap>

<!-- 查询 findUsers 的入参 #{status} 放在结果集上作为查询 findRoleById 作为入参 -->
<select id="findUsers" parameterType="Integer" resultMap="SysUserMap">
    SELECT su.id, su.login_name, su.user_name, su.email, sur.role_id, #{status} AS role_status
    FROM ai_user su
    LEFT JOIN dc_sys_user_role sur ON sur.user_id = su.id
</select>


<select id="findRoleById" resultType="com.answer.ai.entity.SysRole">
    SELECT id, role_name, role_descript AS role_desc
    FROM ai_user_role
    WHERE user_id = #{userId} AND role_status = #{roleStatus}
</select>
```

### 接口调用结果

```
{
    "code": 10000,
    "desc": "success",
    "data": [
        {
            "id": 1,
            "loginName": "admin",
            "userName": "林**",
            "email": "answer_ljm@163.com",
            "roleIds": [
                1
            ],
            "roles": [
                {
                    "id": 1,
                    "roleName": "admin",
                    "roleDesc": "超级管理员"
                }
            ]
        },
        {
            "id": 2,
            "loginName": "answer",
            "userName": "杨**",
            "email": "1072594307@qq.com",
            "roleIds": [
                2,
                3
            ],
            "roles": [
                {
                    "id": 2,
                    "roleName": "answer",
                    "roleDesc": "普通管理员"
                },
                {
                    "id": 3,
                    "roleName": "develop",
                    "roleDesc": "开发者权限"
                }
            ]
        }
    ]
}
```






# 3.参考

MyBatis多层Collection集合嵌套数据返回:

```
https://blog.csdn.net/qq_22067469/article/details/88094722
```




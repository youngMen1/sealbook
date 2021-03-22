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
————————————————
版权声明：本文为CSDN博主「KyrieCao」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_22067469/article/details/88094722
```





# 2.参考
MyBatis多层Collection集合嵌套数据返回:


```
https://blog.csdn.net/qq_22067469/article/details/88094722
```






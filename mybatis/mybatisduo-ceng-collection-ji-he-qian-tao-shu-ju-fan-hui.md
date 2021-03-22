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




# 2.参考
MyBatis多层Collection集合嵌套数据返回:


```
https://blog.csdn.net/qq_22067469/article/details/88094722
```






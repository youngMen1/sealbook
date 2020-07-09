# 1.踩坑日志

## 1.1.短信问题

这两天在阿里云部署项目，项目主体是springboot2.2 + vue2.6.10，本地使用阿里大于短信功能正常，打成jar包，docker镜像，使用阿里云容器服务部署后，短信功能失效。  
我们使用的阿里云服务区域是深圳。**（很重要，短信设置跟区域有关）**

**本地配置**

```
// 依赖部分
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>4.3.8</version>
</dependency>
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-dysmsapi</artifactId>
    <version>1.1.0</version>
</dependency>


// 代码部分（关键的部分）
CommonRequest request = new CommonRequest();
request.setDomain(“dysmsapi.aliyuncs.com”);
request.putQueryParameter("RegionId", "cn-hangzhou");
```




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
request.putQueryParameter("RegionId", "cn-shengzheng");
```

本地运行发送短信功能正常，一旦部署到阿里云，就出现如下错误：（请求超时）

com.aliyuncs.exceptions.ClientException: SDK.ServerUnreachable : Server unreachable: java.net.ConnectException: Connection timed out \(Connection timed out\)

**解决方案**

经多次尝试，该问题得以解决：

[https://help.aliyun.com/document\_detail/68360.html?spm=a2c4g.11186623.6.699.1c0137edsk3wUh](https://help.aliyun.com/document_detail/68360.html?spm=a2c4g.11186623.6.699.1c0137edsk3wUh)



如我的服务区域是上海，我就选择上海的内网IP：

dysmsapi-vpc.cn-shanghai.aliyuncs.com




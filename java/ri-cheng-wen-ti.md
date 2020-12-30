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

```
com.aliyuncs.exceptions.ClientException: SDK.ServerUnreachable : Server unreachable: connection http://dysmsapi.aliyuncs.com/?PhoneNumbers=19909291246&Action=SendSms&Timestamp=2020-07-09T09%3A02%3A17Z&SignName=%E5%8F%A4%E5%BE%B7%E8%8F%B2%E5%8A%9B%E5%81%A5%E8%BA%AB&TemplateCode=SMS_161380087&SignatureVersion=1.0&Format=JSON&SignatureNonce=df5d70ec-49d1-48f8-bf05-58b2d28ba848159428533767359&Version=2017-05-25&AccessKeyId=LTAIE4ZyHMoneEay&Signature=X71UGGkEPE5Tlb3Wn8aWtusk2%2BA%3D&SignatureMethod=HMAC-SHA1&TemplateParam=%7B%22code%22%3A%22790562%22%7D&RegionId=cn-shenzhen failed
```

![](/static/image/微信截图_20200709172936.png)

**解决方案**

1.经多次尝试，该问题得以解决：



```
https://help.aliyun.com/document\_detail/68360.html?spm=a2c4g.11186623.6.699.1c0137edsk3wUh
```



**如我的服务区域是深圳，我就选择深圳的内网IP：**

**dysmsapi-vpc.cn-shenzhen.aliyuncs.com**

2.修改代码如下：

```
CommonRequest request = new CommonRequest();
// 设置区域为对应的内网地址
request.setDomain(“dysmsapi-vpc.cn-shenzhen.aliyuncs.com”);
// 改为深圳
request.putQueryParameter("RegionId", "cn-shanghai");
```
## 1.2.日期格式化使用 YYYY-MM-dd 的潜在问题（正确的写法是yyyy-MM-dd）
我们先来写个单元测试，重现一下这个问题。
测试逻辑：
* 1、创建两个日期格式化，一个是出问题的YYYY-MM-dd，**另一个是正确用法yyyy-MM-dd**
* 2、分别去格式化两个不同的日期：2020年12月26日（周六），2020年12月27日（周日）
具体代码如下：


```

```



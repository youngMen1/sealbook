## SourceTree3.3.8企业版

### 首次点击msi进行安装 （当前最新版SourcetreeEnterpriseSetup\_3.3.8.msi）

![](/static/image/16496299-cd7cb57ba43ac72c.webp)  
![](/static/image/16496299-6798c28cc45930b3.webp)  
![](/static/image/16496299-a7378f6cbe703f9b.webp)  
![](/static/image/16496299-87506f3ae3cb1b25.webp)  
![](/static/image/16496299-cf9c5eb5cb53fb5f.webp)

### 然后找到在 %programfiles\(x86\)%\Atlassian\Sourcetree 目录下找到SourceTree.exe 运行后会出现注册界面

![](/static/image/16496299-f943b4b3c51db332.webp)

### 关掉sourcetree 打开%LocalAppData%\Atlassian\SourceTree

新建文件：accounts.json

```
[
  {
    "$id": "1",
    "$type": "SourceTree.Api.Host.Identity.Model.IdentityAccount, SourceTree.Api.Host.Identity",
    "Authenticate": true,
    "HostInstance": {
      "$id": "2",
      "$type": "SourceTree.Host.Atlassianaccount.AtlassianAccountInstance, SourceTree.Host.AtlassianAccount",
      "Host": {
        "$id": "3",
        "$type": "SourceTree.Host.Atlassianaccount.AtlassianAccountHost, SourceTree.Host.AtlassianAccount",
        "Id": "atlassian account"
      },
      "BaseUrl": "https://id.atlassian.com/"
    },
    "Credentials": {
      "$id": "4",
      "$type": "SourceTree.Model.BasicAuthCredentials, SourceTree.Api.Account",
      "Username": "username@email.com"
    },
    "IsDefault": false
  }
]
```
如图：

![](/static/image/16496299-f8e98d62abce4343.webp

)


## 参考

[https://www.jianshu.com/p/625824c067e6](https://www.jianshu.com/p/625824c067e6)  
[https://www.cnblogs.com/fisherbook/p/11397168.html](https://www.cnblogs.com/fisherbook/p/11397168.html)


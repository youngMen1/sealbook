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

* 1.创建两个日期格式化，一个是出问题的YYYY-MM-dd，**另一个是正确用法yyyy-MM-dd**
* 2.分别去格式化两个不同的日期：2020年12月26日（周六），2020年12月27日（周日）
  具体代码如下：

```
public class Tests {
  @Test 
  public void test() throws Exception { 
    SimpleDateFormat df1 = new SimpleDateFormat("YYYY-MM-dd"); 
    SimpleDateFormat df2 = new SimpleDateFormat("yyyy-MM-dd"); 

    Calendar c = Calendar.getInstance(); 

    // 2020年12月26日周六 
    c.set(Calendar.DATE, 26); 
    System.out.println("YYYY-MM-dd = " + df1.format(c.getTime())); 
    System.out.println("yyyy-MM-dd = " + df2.format(c.getTime())); 

    // 分割线 
    System.out.println("========================"); 
    // 2020年12月27日 周日 
    c.add(Calendar.DATE, 1); 
    System.out.println("YYYY-MM-dd = " + df1.format(c.getTime())); 
    System.out.println("yyyy-MM-dd = " + df2.format(c.getTime())); 
  } 
}
```

跑一下测试，可以看到输出结果如下：

```
YYYY-MM-dd = 2020-12-26 
yyyy-MM-dd = 2020-12-26 
======================== 
YYYY-MM-dd = 2021-12-27 
yyyy-MM-dd = 2020-12-27
```

* 2020年12月26日（周六），两种格式化都正确
* 2020年12月27日（周日），YYYY-MM-dd出了问题，年份到了2021年

**问题原因**

为什么YYYY-MM-dd格式化2020年12月27日的时候，会到2021年呢？

**因为YYYY是week-based-year，表示：当天所在的周属于的年份，一周从周日开始，周六结束，只要本周跨年，那么这周就算入下一年。**

所以2020年12月27日那天在这种表述方式下就已经到 2021 年了。

**而当使用yyyy的时候，就还是 2020 年。**

# 2.Mysql异常

## 2.1.MySql Host is blocked because of many connection errors 解决方法

![](/static/image/微信截图_20200706141311.png)

```
message from server: "Host '192.168.1.28' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'"
```

**原因：**  
同一个ip在短时间内产生太多（超过mysql数据库max\_connection\_errors的最大值）中断的数据库连接而导致的阻塞；  
**解决方法：**

```
1、提高允许的max_connect_errors数量（这种方法不彻底，后期还可能导致异常出现）：

进入Mysql数据库查看max_connect_errors： show variables like ‘max_connect_errors’;

修改max_connect_errors的数量为1000： set global max_connect_errors = 1000;

查看是否修改成功：show variables like 'max_connect_errors';

2、使用mysqladmin flush-hosts 命令清理一下hosts文件

whereis mysqladmin查找mysqladmin的路径

使用命令修改：

/usr/bin/mysqladmin flush-hosts -h192.168.1.121 -uroot -p

备注：

配置有master/slave主从数据库的要把主库和从库都修改一遍

第二步也可以在数据库中进行，命令如下：flush hosts;
mysql> flush hosts;

没有权限的用户先授权或者用root操作

grant all privileges on *.* to 'wang'@'%' identified by 'MyNewPass4!';
```

# 2.2.解决mysql数据库表锁死（表打不开，也关不上）

原因：两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待。

**解决方案：**  
1.查询所有进程

```
show full processlist
```

2.关闭锁死进行，kill + id

```
KILL 168;
KILL 172;
KILL 174;
KILL 177;
```

![](/static/image/20190627135753776.png)  
或者重启mysql,检查造成死锁的代码。

# 2.3.本地替换zookeeper


```
elasticjob:
  regCenter:
    serverLists: localhost:2181
    namespace: udream-elastic-job
```



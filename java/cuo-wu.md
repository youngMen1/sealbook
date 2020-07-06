# Springboot启动异常总结
## 1.1.Intellij IDEA运行报Command line is too long解法

![img](/assets/import.png)

```
<property name="dynamic.classpath" value="true"/>
```

## 1.2.SpringBoot启动异常

![](/assets/springboot启动异常.png)
# Mysql异常


```
message from server: "Host '192.168.1.28' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'"
```
原因：

同一个ip在短时间内产生太多（超过mysql数据库max_connection_errors的最大值）中断的数据库连接而导致的阻塞；



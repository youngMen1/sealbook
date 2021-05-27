# 1.Springboot启动异常总结

## 1.1.Intellij IDEA运行报Command line is too long解法

![img](/assets/import.png)

```
<property name="dynamic.classpath" value="true"/>
```

## 1.2.SpringBoot启动异常

![](/assets/springboot启动异常.png)

## 1.3.Springboot报错说 Failed to parse multipart servlet request; nested exception is java.io.IOException

**原因:**  
1.该异常是如何产生的

我是通过gentman，发送一个post请求，导致该异常的。从上面的异常信息来看，是因为该目录\[/tmp/tomcat.1428942566812653608.8090/work/Tomcat/localhost/ROOT\]，不存在导致的。

**解决方案:**  
1.重启你的项目就可以了（我采用的这种）  
2.在application.yml文件中设置multipart location ，并重启项目

```
spring:
  http:
    multipart:
      location: /data/upload_tmp
```

## 1.4.Idea SpringBoot工程提示 "Error running 'xxxx'": Command line is too long... 问题解决

解决办法：在.idea文件夹下面的workspace.xml中的

```
<component name="PropertiesComponent">
```

标签下面添加：

```
<property name="dynamic.classpath" value="true" />
```




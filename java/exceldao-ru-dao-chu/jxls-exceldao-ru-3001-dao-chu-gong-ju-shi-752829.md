# 1.JXLS (Excel导入、导出工具使用)

 jxls是一个简单的、轻量级的excel导出库，使用特定的标记在excel模板文件中来定义输出格式和布局。java中成熟的excel导出工具有pol、jxl，但他们都是使用java代码的方式来导出excel，编码效率很低且不方便维护。

还可以使用一些工具很轻松的实现模板导出。这些工具现在还在维护，而且做得比较好的国内的有easyPOI，国外的就是这个JXLS了。

比较：

项目中有很多复杂的报表（大量单元格合并和单元格样式），easyPOI处理合并单元格时候容易出现残损的情况，poi代码维护成本高。

```
<dependency>
    <groupId>org.jxls</groupId>
    <artifactId>jxls</artifactId>
    <version>[2.6.0-SNAPSHOT,)</version>
</dependency>
<dependency>
    <groupId>org.jxls</groupId>
    <artifactId>jxls-poi</artifactId>
    <version>[1.2.0-SNAPSHOT,)</version>
</dependency>
<dependency>
    <groupId>org.jxls</groupId>
    <artifactId>jxls-jexcel</artifactId>
    <version>[1.0.8,)</version>
</dependency>
<dependency>
    <groupId>org.jxls</groupId>
    <artifactId>jxls-reader</artifactId>
    <version>[2.0.5,)</version>
</dependency>
```

# 




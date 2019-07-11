# Docker学习-搭建ELK环境整合SpringBoot

# 一、前提

linux安装docker

# 二、Docker安装部署

1、自定义网络

![](file://E:/sealbook/static/image/微信截图_20190711143115.png?lastModify=1562836328)

2、安装启动Elasticsearch

![](file://E:/sealbook/static/image/微信截图_20190711142807.png?lastModify=1562836328 "微信截图\_20190711142807")

3、安装启动Kinaba

第一次失败了，可能是网络原因。

![](file://E:/sealbook/static/image/微信截图_20190711143902.png?lastModify=1562836328 "微信截图\_20190711143902")

4、安装启动Logstash 下载镜像

![](file://E:/sealbook/static/image/微信截图_20190711144236.png?lastModify=1562836328 "微信截图\_20190711144236")

复制logstash容器配置文件到主机

![](file://E:/sealbook/static/image/微信截图_20190711144519.png?lastModify=1562836328 "微信截图\_20190711144519")

停止删除容器

```
docker kill logstashdocker rm logstash
```

修改主机/usr/share/logstash/pipeline/logstash.conf文件，如下

```
input {
tcp {
port =
>
 5000
}}## Add your filters / logstash plugins configuration hereoutput {
elasticsearch {
hosts =
>
 "elasticsearch:9200" 
index =
>
 "logstash-%{+YYYY.MM.dd}"
}}
```

可以先用一下命令启动看能否启动成功，访问主机ip：9600 验证

```
docker run --rm -it --name=logstash --net=elk -p 9600:9600 -p 5000:5000 -v /usr/share/logstash/config/:/usr/share/logstash/config/ -v /usr/share/logstash/pipeline/:/usr/share/logstash/pipeline/ logstash:6.5.4
```

服务器内存不足

参考:[https://github.com/elastic/elasticsearch/issues/22245](https://github.com/elastic/elasticsearch/issues/22245)

有一个名为jvm.options的文件`/etc/elasticsearch/jvm.options`，只需添加`-XX:+AssumeMP`或者-XX:-AssumeMP到此文件中。

![](file://E:/sealbook/static/image/微信截图_20190711164934.png?lastModify=1562836328 "微信截图\_20190711164934")

后台启动

```
docker run -d --name=logstash --net=elk -p 9600:9600 -p 5000:5000 -v /usr/share/logstash/config/:/usr/share/logstash/config/ -v /usr/share/logstash/pipeline/:/usr/share/logstash/pipeline/ -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime logstash:6.5.4
```

主机/usr/share/logstash/config/、/usr/share/logstash/pipeline/文件夹映射到logstash容器/usr/share/logstash/config/、/usr/share/logstash/pipeline/文件夹

这里还遇到个很坑的问题，linux主机跟docker容器里面的时间都不准，导致logstash收集的日志时间不对

首先校正Linux主机时间，参考[https://www.cnblogs.com/zhi-leaf/p/6281549.html](https://www.cnblogs.com/zhi-leaf/p/6281549.html)

同步主机时间到Docker容器，启动加`-v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime`，参考[https://blog.csdn.net/dounine/article/details/79976778](https://blog.csdn.net/dounine/article/details/79976778)

也可以选择使用非官方的ELK镜像，直接搭建ELK环境。

# 三、SpringBoot、Logstash简单整合

所谓ELK，就是Elasticsearch存储，Kibana展示，Logstash收集，这里给出一个Springboot输出日志到Logstash的简单配置

application.yml

```
logging:  config: classpath:logback-spring.xml
```

logback-spring.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <property name="LOGSTASH_HOST" value="${LOGSTASH_HOST:-${DOCKER_HOST:-192.168.244.137}}"/>
    <property name="LOGSTASH_PORT" value="${LOGSTASH_PORT:-5000}"/>
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>${LOGSTASH_HOST}:${LOGSTASH_PORT}</destination>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"appname":"myapp"}</customFields>
        </encoder>
    </appender>
    <springProfile name="default,dev,test">>
        <logger name="io.vickze" level="DEBUG">
            <appender-ref ref="LOGSTASH"/>
        </logger>
    </springProfile>
    <springProfile name="prod">
        <logger name="io.vickze" level="ERROR">
            <appender-ref ref="LOGSTASH"/>
        </logger>
    </springProfile>
</configuration>
```

具体代码可参考

本文只写了logstash tcp为input，output为elasticsearch的搭建与应用，logstash本身支持的input、output有几十种，具体可参考


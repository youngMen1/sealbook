# 1.JProfiler是什么 {#1}

JProfiler是由ej-technologies GmbH公司开发的一款性能瓶颈分析工具\(该公司还开发部署工具\)。  
其特点:

* 使用方便
* 界面操作友好
* 对被分析的应用影响小
* CPU,Thread,Memory分析功能尤其强大
* 支持对jdbc,noSql, jsp, servlet, socket等进行分析
* 支持多种模式\(离线，在线\)的分析
* ![img](/static/image/f71a75090d48e46eb809001918d37d7cc8d5ec90.png)
* 跨平台

## 二.数据采集 {#2}

Q1. JProfiler既然是一款性能瓶颈分析工具，这些分析的相关数据来自于哪里？  
Q2. JProfiler是怎么将这些数据收集并展现的?
![img](/static/image/774e1de366c3dced5bf97ab0cd34471ec9a99537.png)
(图2)

A1. 分析的数据主要来自于下面俩部分
1. 一部分来自于jvm的分析接口**JVMTI**(JVM Tool Interface) , JDK必须>=1.6


```
JVMTI is an event-based system. The profiling agent library can register handler functions for different events. It can then enable or disable selected events
```



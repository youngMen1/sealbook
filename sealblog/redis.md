# 一、centos7安装redis

## 第一步：下载redis安装包

```
[root@iZwz991stxdwj560bfmadtZ local]# wget http://download.redis.io/releases/redis-4.0.6.tar.gz
--2017-12-13 12:35:12--  http://download.redis.io/releases/redis-4.0.6.tar.gz
Resolving download.redis.io (download.redis.io)... 109.74.203.151
Connecting to download.redis.io (download.redis.io)|109.74.203.151|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1723533 (1.6M) [application/x-gzip]
Saving to: ‘redis-4.0.6.tar.gz’

100%[==========================================================================================================>] 1,723,533    608KB/s   in 2.8s   

2017-12-13 12:35:15 (608 KB/s) - ‘redis-4.0.6.tar.gz’ saved [1723533/1723533]
```

## 第二步：解压压缩包
```
[root@iZwz991stxdwj560bfmadtZ local]# tar -zxvf redis-4.0.6.tar.gz
```
## 第三步：yum安装gcc依赖
```
[root@iZwz991stxdwj560bfmadtZ local]# yum install gcc
遇到选择,输入y即可
```
## 第四步：跳转到redis解压目录下
```
[root@iZwz991stxdwj560bfmadtZ local]# cd redis-4.0.6
```
## 第五步：编译安装
```
[root@iZwz991stxdwj560bfmadtZ redis-4.0.6]# make MALLOC=libc
```
将/usr/local/redis-4.0.6/src目录下的文件加到/usr/local/bin目录
```
[root@iZwz991stxdwj560bfmadtZ redis-4.0.6]# cd src && make install
    CC Makefile.dep

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
```
# 二、启动redis的三种方式
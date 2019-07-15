[https://www.cnblogs.com/javajetty/p/9954685.html](https://www.cnblogs.com/javajetty/p/9954685.html)

# Zookeeper-3.4.14

# 创建安装目录、并下载安装包

```
# mkdir -p /usr/local/zookeeper
# cd /usr/local/zookeeper
# wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
```

![](/assets/微信截图_20190715150826.png)

# 解压安装包

```
tar -zxvf zookeeper-3.4.14.tar.gz
```

# 添加系统环境变量 {#3添加系统环境变量}

```
输入vim /etc/profile命令打开系统配置文件，并在其尾部追加如下内容：

export ZOOKEEPER_HOME=/usr/local/zookeeper/zookeeper-3.4.14/
export PATH=$ZOOKEEPER_HOME/bin:$PATH
export PATH

#保存强制退出
:wq!

输入 source /etc/profile命令使其文件立即生效
```

### 修改zookeeper配置文件 {#4修改zookeeper配置文件}

将配置文件`zoo_sample.cfg` 复制为 `zoo.cfg`，操作如下：

```
cd /usr/local/zookeeper/zookeeper-3.4.14/conf/
cp zoo_sample.cfg zoo.cfg
```




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

使用

`vim zoo.cfg`

命令打开，并修改其内容如下：

```
# 服务器与客户端之间交互的基本时间单元（ms）

tickTime=2000

# 配置保存数据文件夹

dataDir=/usr/local/zookeeper/zookeeper-3.4.14/data

# 配置保存日志文件夹，当此配置不存在时默认路径与dataDir一致

dataLogDir=/usr/local/zookeeper/zookeeper-3.4.14/logs

# 客户端访问zookeeper的端口号

clientPort=2181
```

### 启动、关闭zookeeper服务 {#5启动关闭zookeeper服务}

```
启动服务：zkServer.sh start
查看状态：zkServer.sh status
关闭服务：zkServer.sh stop
重启服务：zkServer.sh restart
```

![](/assets/微信截图_20190715152015.png)

启动成功

![](/assets/微信截图_20190715155455.png)

![](/assets/微信截图_20190715161045.png)

# 注意：

如果查看状态：zkServer.sh status启动失败

![](/assets/微信截图_20190715155637.png)

查看zookeeper.out文件:nohup: failed to run command ‘/usr/java/jdk1.8.0\_181/bin/java’: No such file or directory

```
# 查看系统环境变量
export
# 修改系统环境变量
export JAVA_HOME=/usr
```

**再次查看状态**

\[work@xxx zookeeper-3.4.14\]$ bin/zkServer.sh status  
JMX enabled by default  
Using config: /home/work/soft/zookeeper-3.4.14/bin/../conf/zoo.cfg  
Mode: standalone

[https://www.cnblogs.com/abc-begin/p/8206727.html](https://www.cnblogs.com/abc-begin/p/8206727.html)


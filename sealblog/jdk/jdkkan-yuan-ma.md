# 1.基本介绍

在学习JDK源码的时候，自然少不了代码的调试。

阅读与调试各个版本JDK 的环境搭建基本一致，这里以JDK1.8为例。

1.1.首先，在安装的jdk1.8路径下，找到src.zip和javafx-src.zip压缩文件

微信截图\_20200426161523.png

1.2.选择一个合适的目录 复制过来一份\(我直接放在项目jdk8目录下\)

微信截图\_20200426161704.png

1.3.新建一个sdk

新建项目完成之后，点击file&gt;project structure,然后选中SKDS，切换到Sourcepath选项

将原先的src.zip和javafx-src.zip依赖，“-”减号删去，“+”好新建你本地**解压后的src和javafx-src依赖**，之后我们点击apply

微信截图\_20200426161814.png

1.4.然后点击file --&gt; Setting --&gt; Build,Execution,Deployment --&gt; Debugger --&gt; Stepping把Do not step into the classes中的ajva.\*，javax.\*取消勾选，其他的随意, 点击apply。

微信截图\_20200426161917.png

# 2.总结

**源码的注释全部在javafx-src里面**

微信截图\_20200426162007.png


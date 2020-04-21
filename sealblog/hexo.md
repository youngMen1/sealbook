# 使用Hexo搭建个人Github博客

参考网上的文章总算把自己的Github博客搭建出来了，在这把我的搭建步骤分享给大家，后面的内容还包括了配置域名，如已搭建成功了想要使用自己的域名访问博客可以直接跳到配置域名部分。[搭建个人博客-hexo+github详细完整步骤](http://www.jianshu.com/p/189fd945f38f)[零基础免费搭建个人博客-hexo+github](http://blog.csdn.net/jzooo/article/details/46781805)上面两个链接是我所参考的文章，写的也很详细，如果没看懂我的步骤也可以看看上面的。

# 一、准备

我们需要安装Git、Node.js、Hexo以及注册一个GitHub账号。下载Git、Nodejs可以选择在官网下载，也可以去CSDN下载，大部分都是不需要积分的。PS：官网下载网速超级慢，不知道是资源问题还是墙的原因。[Git官网下载地址](https://git-for-windows.github.io/)[Node.js官网下载地址](https://nodejs.org/en/)[Git CSDN下载](http://download.csdn.net/search/0/10/0/2/1/Git 2.13.0)[Node.js CSDN下载](http://download.csdn.net/search/0/10/0/2/1/Node.js v6.10.3)

## 1、安装Git

![](https://blog.yk95.top/static/img/2017-5-25_22-21-15.png)

打开Git安装程序，点击NEXT来到这个页面，选择要安装的组件，可以全选也可以默认，然后一路NEXT即可，安装路径根据自己习惯更改。

## 2、安装Node.js

![](https://blog.yk95.top/static/img/2017-5-25_22-33-36.png)同样打开Node.js安装程序，一路默认即可，安装路径根据自己习惯更改。

## 3、安装Hexo

安装Hexo就稍微繁琐点，不过大家一定不能急，耐心等待安装，一般来说按照步骤慢慢来都是没有问题的。首先在任意地方右键，点击“Git Bash Here”。![](https://blog.yk95.top/static/img/2017-5-25_22-38-12.png)使用NPM命令安装，为防止被墙，这里使用淘宝NPM镜像，输入命令`npm install -g cnpm --registry=https://registry.npm.taobao.org`等待安装完成。

![](https://blog.yk95.top/static/img/2017-5-22_21-28-49.png)完成后，继续输入命令`cnpm install -g hexo-cli`

![](https://blog.yk95.top/static/img/2017-5-22_21-29-39.png)

等待完成，再输入命令`cnpm install hexo --save`

![](https://blog.yk95.top/static/img/2017-5-22_21-29-57.png)

至此Hexo安装完成，使用查看版本命令`hexo -v`检查是否正常安装。

![](https://blog.yk95.top/static/img/2017-5-22_21-30-53.png)

## 4、注册Github以及创建仓库

接下来我们注册Github账号，使用常用邮箱注册即可，过程比较简单这里就不细讲了。注册成功登录后，来到我的仓库页面，点击New repository。

![](https://blog.yk95.top/static/img/2017-5-27_22-31-44.png)

注意Repository name一定得是yourname.github.io，这样才能使用这个地址访问到你的Github page，填好Repository name，点击Create repository。（我这里因为之前创建过，所以报同名错误，大家第一次创建的话可以忽略）。

![](https://blog.yk95.top/static/img/2017-5-27_22-37-26.png)

在[零基础免费搭建个人博客-hexo+github](http://blog.csdn.net/jzooo/article/details/46781805)里有个‘启用GitHub Page’的步骤，但我发现页面都已经变得不一样了，最新的页面如下所示，只需要Choose Theme就会自动启用Github Page。

![](https://blog.yk95.top/static/img/2017-5-27_23-46-36.png)

创建仓库后我们后面的步骤需要用到仓库地址，进到yourname.github.io仓库页面，看下图。

![](https://blog.yk95.top/static/img/2017-5-27_22-53-59.png)

# 二、本地启动与部署到Github

## 1、本地启动

创建一个新文件夹（我的是在E盘创建的Blog），进入该文件夹，右键Git Bash Here，输入`hexo init`命令。PS：由于博主搭建成功后并没有推倒再来一遍，所以到这里就没有截图了，大家键入命令后，在程序运行过程记得一定要耐心等待。

初始化成功后，大概是下面的目录结构（我这个是部署到git后，有多了几个文件）。

![](https://blog.yk95.top/static/img/2017-5-27_23-8-46.png)

接下来输入`hexo s -g`命令启动，启动后浏览器访问localhost:4000查看博客效果。

## 2、部署到Github

本地成功后下面就要部署到Git了，打开\_config.yml进行配置，如下图，复制你的仓库地址给repo参数\(上面有讲怎么复制）。

![](https://blog.yk95.top/static/img/2017-5-27_23-21-41.png)

在Git命令窗口输入`npm install hexo-deployer-git --save`安装hexo-deployer-git自动部署发布工具，等待安装完成，输入`hexo clean && hexo g && hexo d`命令发布到Github，这里注意第一次发布的话会需要输入你的Github账号跟密码，等待出现下图的信息就说明发布成功了，在浏览器输入yourname.github.io就可以看到你的博客了。

![](https://blog.yk95.top/static/img/2017-5-27_23-33-28.png)

# 三、选择主题与配置域名

## 1、选择主题

完成上面的步骤之后呢，可能有人会觉得默认的Hexo主题不是特别好看（至少博主是那么认为的），所以我们可以给博客选择一个适合自己的主题，使用命令`git clone https://github.com/iissnan/hexo-theme-nextthemes/[theme]`来下载一个新的主题，\[theme\]为主题名。下载完成后，修改\_config.yml的theme参数来配置主题，见下图。

![](https://blog.yk95.top/static/img/2017-5-29_16-24-31.png)

附上链接：[有哪些好看的 Hexo 主题？](https://www.zhihu.com/question/24422335)

博主选择的主题是yilia，这里遇到了一个坑：使用yilia主题有了两个\_config.yml文件，一个是我们一直用到的，另一个是yilia主题目录下的，启用yilia的某些功能需要在我们一直用到的\_config.yml文件配置，而yilia主题的定制是在yilia目录下的\_config.yml配置，其他主题可能也会有这样的情况，这一点稍微注意下。

另附上yilia主题的评论配置：[多说、畅言、网易云跟帖、Disqus评论配置](https://github.com/litten/hexo-theme-yilia/wiki/多说、畅言、网易云跟帖、Disqus评论配置)

## 2、配置域名

这一步骤提供给需要使用自己的域名访问Github page的读者，不需要的可以直接跳过。在cmd窗口使用`ping yourname.github.io`得到IP地址，见下图。

![](https://blog.yk95.top/static/img/2017-5-29_16-0-36.png)

在你的Github博客仓库根目录下创建CNAME文件，注意不能有文件名不能有后缀且要大写，内容为你想指定的域名。

![](https://blog.yk95.top/static/img/2017-5-29_16-11-59.png)

然后将你的域名映射到该IP地址，这里以博主的阿里云购买的域名举例，在阿里云域名控制台添加一条解析，如下图。

![](https://blog.yk95.top/static/img/2017-5-29_16-18-31.png)

等解析生效就可以使用域名访问Github page了，例如博主的：[http://yk95.top](http://yk95.top/)

使用域名访问Github page还需要注意一点，我们使用`hexo clean && hexo g && hexo d`命令将博客发布到Git时，Hexo会将整个仓库全部清空，然后才提交，这样我们创建的CNAME文件就被删除了，这里提供一个简单的解决方案，在本地博客public文件夹下创建CNAME文件，发布到Git时不clean使用`hexo g && hexo d`命令，发布时会将CNAME文件一起提交。

# 四、发布自己的第一篇博客

将博客搭建起来之后就可以开始写博客了，首先需要配置一些基本信息，这些内容不会解析到博客正文中，见下图。

![](https://blog.yk95.top/static/img/2017-5-29_16-54-21.png)

接下来就是正式写博客正文了，写的文章要遵循markdown语法。附上链接：[Markdown 语法说明 \(简体中文版\)](http://www.appinn.com/markdown/#img)写好博客后就可以使用命令`hexo clean && hexo g && hexo d`发布到Github了（域名访问的请去掉`hexo clean`），下面是博客效果。

![](https://blog.yk95.top/static/img/2017-5-29_19-55-23.png)

至此，本篇博客搭建教程介绍完毕，最后再附上一些链接：[Hexo博客添加百度sitemap](http://www.jianshu.com/p/ab44b916a8b6)[Hexo插件之百度主动提交链接](http://hui-wang.info/2016/10/23/Hexo插件之百度主动提交链接/)[用阿里云的免费 SSL 证书让网站从 HTTP 换成 HTTPS](https://ninghao.net/blog/4449)


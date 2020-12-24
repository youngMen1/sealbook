# Git基础

## 1.安装Git

Git 官方文档地址：`https://git-scm.com/book/zh/v2`  
macOS 平台 Git 下载地址：`https://git-scm.com/download/mac`  
Windows 平台 Git 下载地址：`https://git-scm.com/download/win`  
Linux 平台 Git 下载地址：`https://git-scm.com/download/linux`

## 2.使用Git之前需要做的最小配置

### 2.1.配置User提交作者信息:

Mac:

```
git config --local user.name 'lunzi'
git config --local user.email 'lunzi@163.com'
```

Windows:

```
git config --local user.name "lunzi"
git config --local user.email "lunzi@163.com"
```

### 2.2.参数区别:

git config --local \#\#只对某个仓库有效,切换到另外一个仓库失效  
git config --global \#\#当前用户的所有仓库有效,工作当中最常用  
git config --system \#\#系统的所有用户,几乎不用

### 2.3.查看配置:

git config --list --local \#\#只能在仓库里面起作用, 普通路径git不管理  
git config --list --global  
git config --list --system

local的在.git/config里面；  
global的在个人home目录下的.gitconfig里面；  
system应该在git安装目录的下;

### 2.4.config 增删改查操作

新增：`git config --global --add configname configvalue`  
删除：`git config --global --unset configname`  
修改：`git config --global configname configvalue`  
查询：`git config --global configname`


## 3.创建第一个仓库并配置用户信息
### 3.1.在F:\git\repositories盘的git目录下创建
进入操作目录：`cd f:/git`
新建文件夹：`mkdir git_learning`
进入新建的文件夹：`cd git_learning`
转化为git版本库：`git init`—将所在目录（git_learning）转化为git版本库（此时的版本库是空的，目录下的文件默认不会被放入版本库中，视为临时文件）

或者
进入目录创建`git_learning`文件夹: `git init git_learning`





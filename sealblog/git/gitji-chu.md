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

方式一：  
进入操作目录：`cd f:/git`  
新建文件夹：`mkdir git_learning`  
进入新建的文件夹：`cd git_learning`  
转化为git版本库：`git init`—将所在目录（git\_learning）转化为git版本库（此时的版本库是空的，目录下的文件默认不会被放入版本库中，视为临时文件）

方式二：

进入目录创建`git_learning`文件夹: `git init git_learning`

### 3.2.将文件添加到版本库中

将index.html（指定文件）添加到版本库中：git add index.html  
将当前目录及子目录中的文件都添加到版本库里：git add

### 3.3.提交文件

提交文件（提交先要进行git add操作\)：`git commit -m "提交新文件"`  
合并git add 和 git commit 操作（适用于比较小的变更\): `git commit -am "提交信息"`

## 4.通过几次commit来认识工作区和暂存区

### 4.1.`git add .` 和 `git add -u` 区别

```
git add . ：将工作空间新增和被修改的文件添加的暂存区
git add -u :将工作空间被修改和被删除的文件添加到暂存区(不包含没有纳入Git管理的新增文件)  

# 创建仓库
git init [project folder name]  初始化 git 仓库
git add [fileName]  把文件从工作目录添加到暂存区
git commit -m'some information'  用于提交暂存区的文件
git commit -am'Some information' 用于提交跟踪过的文件
git log  查看历史
git status  查看状态
```

git add -u 可以添加所有已经被 git 控制的文件到暂存区
以前删除文件夹只会用 「-rf」，今天学到了 「-r」，并得知它们两个区别：「-r」 有时候会提示是否确认删除。


**课程中使用的几个命令，对应windows版：**
```
# 清屏
Mac: clear
Windows: cls

# 展示文件夹内的内容
Mac: ls -al
Windows: dir

# 展示文件内容（比如展示style.css的内容）
Mac: vi style.css
Windows: more style.css
```




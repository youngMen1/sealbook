# Git基础

## 1.安装Git
Git 官方文档地址：`https://git-scm.com/book/zh/v2`
macOS 平台 Git 下载地址：`https://git-scm.com/download/mac`
Windows 平台 Git 下载地址：`https://git-scm.com/download/win`
Linux 平台 Git 下载地址：`https://git-scm.com/download/linux`

## 2.使用Git之前需要做的最小配置
### 2.1.配置User信息:
git config --local user.name 'lunzi'
git config --local user.email 'lunzi@163.com'

### 2.2.参数区别:
git config --local ##只对某个仓库有效,切换到另外一个仓库失效
git config --global ##当前用户的所有仓库有效,工作当中最常用
git config --system ##系统的所有用户,几乎不用

### 2.3.查看配置:

git config --list --local ##只能在仓库里面起作用, 普通路径git不管理 
git config --list --global 
git config --list --system 

local的在.git/config里面；global的在个人home目录下的.gitconfig里面；system应该在git安装目录的下
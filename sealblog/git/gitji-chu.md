# Git基础

## 1.安装Git
Git 官方文档地址：`https://git-scm.com/book/zh/v2`
macOS 平台 Git 下载地址：`https://git-scm.com/download/mac`
Windows 平台 Git 下载地址：`https://git-scm.com/download/win`
Linux 平台 Git 下载地址：`https://git-scm.com/download/linux`

## 2.使用Git之前需要做的最小配置
### 2.1.添加User配置

git config [--local | --global | --system] user.name 'Your name'
git config [--local | --global | --system] user.email 'Your email'

### 2.2.查看配置

git config --list [--local | --global | --system]

### 2.3.区别

local：区域为本仓库
global: 当前用户的所有仓库
system: 本系统的所有用户
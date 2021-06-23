# 1.基本介绍

## 1.1.命令总结

| 命令 | 描述 |
| :---: | :---: |
| git push | 提交 |
| git pull | 拉取 |
| git branch -a | 查看所有分支\(这里会进入log，退出ctrl + c不管用，直接输入q即可（或者：q）退出\) |
| git checkout 分支名 | 切换分支 |
| git branch | 查看当前使用分支\(结果列表中前面标\*号的表示当前使用分支\) |
| git checkout -b new-branch | 开启新分支，为了便于演示，我们将新分支命名为new-branch |
| git remote -v | 查看GIT项目是从GIT的哪个分支上拉下来的命令 |
| git stash | 把本地当前改动暂存起来，此时master分支就恢复到了上次拉取时的状态 |
| git stash pop | 将改动pop到自己当前的分支 |
| git reset --soft | 不删除工作空间改动代码，撤销commit，不撤销git add，回退到某个版本，只回退了commit的信息，如果还要提交，直接commit即可 |
| git reset -–hard | 删除工作空间改动代码，撤销commit，撤销git add 注意完成这个操作后，就恢复到了上一次的commit状态。 |

## 2.如何使用？

## 2.1.如何把master分支代码合并到自己的分支

1.首先切换到主分支

```
git checkout master
```

2.使用git pull 把领先的主分支代码pull下来

```
git pull
```

3.切换到自己的分支

```
git checkout xxx(自己的分支)
```

4.把主分支的代码合并\(merge\)到自己的分支

```
git merge master
```

5.git push推上去ok完成,现在 你自己分支的代码就和主分支的代码一样了

```
git push origin 自己分支名
```

## 2.2.git合并某次提交到当前分支

有的时候，在develop分支开发，是大家公用的开发分支，但是只想合并自己提交的到master,如何操作呢？那就要用cherry-pick了

语法 : `git cherry-pick commitid`

首先，git log查看自己提交的log，找到版本号，如最近的版本号是 9caedb2313425a953cb59bf36878f539465e2b20  
例如：

```
git cherry-pick 9caedb2313425a953cb59bf36878f539465e2b20
```

如果想合并多个提交到当前分支  
语法：

```
git cherry-pick <C commit-id> <D commit-id> <E commit-id>
```

例如:`git cherry-pick 9caedb2313425a953cb59bf36878f539465e2b20 e680af9f64960be3c6de002ce410ce38924c74be`

## 2.3.如果你想保留刚才本地修改的代码，并把git服务器上的代码pull到本地（本地刚才修改的代码将会被暂时封存起来）

如果你想保留刚才本地修改的代码，并把git服务器上的代码pull到本地（本地刚才修改的代码将会被暂时封存起来）

```
error: Your local changes to the following files would be overwritten by merge:
```

```
// 备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中。
git stash
git pull
// 从Git栈中读取最近一次保存的内容，恢复工作区的相关内容。
git stash pop
```
## 2.4.可以看到弹框底部有Force Checkout、Don`t checkout、Smart Checkout,表示什么意思呢
Smart Checkout就会把冲突的这部分内容带到开发分支（如果你没有点进窗口的那些文件处理冲突的话),比如我在test分支修改到代码，要切换到master分支，点击smart checkout后，master分支会有test分支修改到代码，最好是选smart checkout这样会把本地修改的代码先保存到statsh中，再checkout分支。

Force Checkout  就不会把冲突的这部分内容带到开发分支，如果点了force checkout则本地修改都会丢失！！！！！！！！！正确操作是： 切换分支之前，应该先GIT --> Repository --> Stash changes 保存该分支下的改动。 切换回来后，GIT --> Repository --> UnStash changes 恢复之前的改动，

 Don`t checkout   当然是不切分支,继续留在当前分支了

总结：不要点击force checkout，如果不想当前分支修改到代码出现在要切换到分支中，需要手动Stash changes，如果允许当前分支修改到代码出现在要切换到分支中，可以选择smart checkout

## 3.参考

**Git 参考手册**

```
http://gitref.justjavac.com/branching/#merge
```

**Git 团队协作开发：**

```
https://juejin.im/post/5ea8daa66fb9a04382227c14
```

git教程：`https://aimuch.com/2019/03/21/git%E6%95%99%E7%A8%8B/#more`

Git push 时如何避免出现 "Merge branch 'master' of ...":

```
https://www.cnblogs.com/yanglang/p/11679858.html
```




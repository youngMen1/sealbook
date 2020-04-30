# 1.基本介绍

## 1.1.命令总结

| 命令 | 描述 |
| :---: | :---: |
| git push | 提交 |
| git pull | 拉取 |
| git branch -a | 查看所有分支 |
| git checkout 分支名 | 切换分支 |
| git branch | 查看当前使用分支\(结果列表中前面标\*号的表示当前使用分支\) |
| git checkout -b new-branch | 开启新分支，为了便于演示，我们将新分支命名为new-branch |
|  |  |

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

4.把主分支的代码merge到自己的分支

```
git merge master
```

5.git push推上去ok完成,现在 你自己分支的代码就和主分支的代码一样了

git push origin 自己分支名

## 参考

[http://gitref.justjavac.com/branching/\#merge](http://gitref.justjavac.com/branching/#merge)

**Git 团队协作开发：**

[https://juejin.im/post/5ea8daa66fb9a04382227c14](https://juejin.im/post/5ea8daa66fb9a04382227c14)


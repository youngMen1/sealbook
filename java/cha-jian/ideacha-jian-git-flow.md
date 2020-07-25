# 1.IDEA插件Git Flow

![](/static/image/1_uUpzVOpdFw5V-tJ_YvgFmA.26e56be6.png)

[Git Flow Integration](https://plugins.jetbrains.com/plugin/7315-git-flow-integration)

## 插件作用

集成 Git Flow让我们更加专注在 开发 这件事上。

## 安装

![](/static/image/image-20200714124809715.9ea8177c.png)

## 使用

最开始还没有初始化的时候，点击右下脚 GitFlow init  
![](/static/image/image-20200714125126171.e3160049.png)  
直接 默认 设置就好，点击 Ok 之后，就可以开始使用了。  
![](/static/image/image-20200714125309887.1434693b.png)  
按照最规范的流程走，可以避免在未来某个阶段掉坑里。

## Git 版本管理规范 【Git Flow】

master：永远处于production-ready状态

* 主分支，产品的功能全部实现后，最终在master分支对外发布；

* 只读分支，只能从release或hotfix分支合并，不能修改；

* 所有在master分支的推送应该做标签记录，方便追溯。

develop：最新的下次发布的开发状态

* 主开发分支，基于master分支克隆，发布到下一个release；
* 只读分支，feature功能分支完成，合并到develop（不推送）；
* develop拉取release分支，提测；
* release/hotfix分支上线完毕，合并到develop并推送。

feature：开发新功能都从develop分支出来，完成后merge回develop

* 功能开发分支，基于develop分支克隆，用于新需求的开发；
* 功能开发完毕后合并到develop分支（未正式上线之前不能推送到远程中央仓库）
* feature可以同时存在多个，用于团队多功能同步开发，属于临时分支，开发完毕后可以删除。

release：准备要release的版本，只修bug。从develop出来，完成后merge回master和develop

* 测试分支，feature分支合并到develop分支之后，从develop分支克隆；
* 只要用于提交给测试人员进行功能测试，测试过程中如果发现BUG在release分支修复，修复完成上线后合并到
* develop/master分支并推送完成，做标签记录；
* 临时分支，上线后可删除。

hotfix：等不及release版本就必须马上修复master上线。从master出来，完成后merge回master和develop

* 补丁分支，基于master分支克隆，主要用于对线上的版本进行BUG修复；
* 修复完毕后合并到develop/master分支并推送，做标签记录；
* 所有hotfix分支的修改会进入到下一个release；
* 临时分支，补丁修复上线后可以删除；




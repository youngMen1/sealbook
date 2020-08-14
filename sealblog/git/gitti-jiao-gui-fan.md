# 1.Git提交规范

commit message应该如何写才更清晰明了？团队开发中有没有遇到过让人头疼的git commit？本文分享在git commit规范建设上的实践，规定了commit message的格式，并通过webhook在提交时进行监控，避免不规范的代码提交。

Git每次提交代码都需要写commit message，否则就不允许提交。一般来说，commit message应该清晰明了，说明本次提交的目的，具体做了什么操作……但是在日常开发中，大家的commit message千奇百怪，中英文混合使用、fix bug等各种笼统的message司空见怪，这就导致后续代码维护成本特别大，有时自己都不知道自己的fix bug修改的是什么问题。基于以上这些问题，我们希望通过某种方式来监控用户的git commit message，让规范更好的服务于质量，提高大家的研发效率。

## 1.1.规范梳理

初期我们在互联网上搜索了大量有关git commit规范的资料，但只有Angular规范是目前使用最广的写法，比较合理和系统化，并且有配套的工具（IDEA就有插件支持这种写法）。最后综合阿里巴巴高德地图相关部门已有的规范总结出了一套git commit规范。

### commit message格式

```
<type>(<scope>): <subject>
```
#### type(必须)

用于说明git commit的类别，只允许使用下面的标识。

feat：新功能（feature）。

fix/to：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。

* fix：产生diff并自动修复此问题。适合于一次提交直接修复问题
* to：只产生diff不自动修复此问题。适合于多次提交。最终修复问题提交时使用fix

docs：文档（documentation）。

style：格式（不影响代码运行的变动）。

refactor：重构（即不是新增功能，也不是修改bug的代码变动）。

perf：优化相关，比如提升性能、体验。

test：增加测试。

chore：构建过程或辅助工具的变动。

revert：回滚到上一个版本。

merge：代码合并。

sync：同步主线或分支的Bug。

#### scope(可选)

scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

例如在Angular，可以是location，browser，compile，compile，rootScope， ngHref，ngClick，ngView等。如果你的修改影响了不止一个scope，你可以使用*代替。


#### subject(必须)

subject是commit目的的简短描述，不超过50个字符。

建议使用中文（感觉中国人用中文描述问题能更清楚一些）。

* 结尾不加句号或其他标点符号。
* 根据以上规范git commit message将是如下的格式：

```
fix(DAO):用户查询缺少username属性 
feat(Controller):用户查询接口开发
```
以上就是我们梳理的git commit规范，那么我们这样规范git commit到底有哪些好处呢？

* 便于程序员对提交历史进行追溯，了解发生了什么情况。
* 一旦约束了commit message，意味着我们将慎重的进行每一次提交，不能再一股脑的把各种各样的改动都放在一个git commit里面，这样一来整个代码改动的历史也将更加清晰。
* 格式化的commit message才可以用于自动化输出Change log。

## 1.2.监控服务

通常提出一个规范之后，为了大家更好的执行规范，就需要进行一系列的拉通，比如分享给大家这种规范的优点、能带来什么收益等，在大家都认同的情况下最好有一些强制性的措施。当然git commit规范也一样，前期我们分享完规范之后考虑从源头进行强制拦截，只要大家提交代码的commit message不符合规范，直接不能提交。但由于代码仓库操作权限的问题，我们最终选择了使用webhook通过发送警告的形式进行监控，督促大家按照规范执行代码提交。除了监控git commit message的规范外，我们还加入了大代码量提交监控和删除文件监控，减少研发的代码误操作。

### 整体流程
![](/static/image/v2-3342b351bec8c267c7fd52d33bdb00c5_720w.png)
* 服务注册：服务注册主要完成代码库相关信息的添加。
* 重复校验：防止merge request再走一遍验证流程。
* 消息告警：对不符合规范以及大代码量提交、删除文件等操作发送告警消息。
* DB：存项目信息和git commit信息便于后续统计commit message规范率。

webhook是作用于代码库上的，用户提交git commit，push到仓库的时候就会触发webhook，webhook从用户的commit信息里面获取到commit message，校验其是否满足git commit规范，如果不满足就发送告警消息；如果满足规范，调用gitlab API获取提交的diff信息，验证提交代码量，验证是否有重命名文件和删除文件操作，如果存在以上操作还会发送告警消息，最后把所有记录都入库保存。

以上就是我们整个监控服务的相关内容，告警信息通过如下形式发送到对应的钉钉群里：
v2-91233f42edba80789ef59afd0198bf7e_720w.png
v2-540621f9fc00d793668d426e9d3ffe0d_720w.png

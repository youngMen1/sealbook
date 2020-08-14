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
![](/static/image/v2-91233f42edba80789ef59afd0198bf7e_720w.png)
![](/static/image/v2-540621f9fc00d793668d426e9d3ffe0d_720w.png)
![](/static/image/v2-30df9c298f12b77d66afd3ac6c8b7311_720w.png)


我们也有整体git commit的统计，统计个人的提交次数、不规范次数、不规范率等如下图：

![](/static/image/v2-aea9b790d15a7c76e9e07282c90c354e_720w.png)

## 1.3.未来思考

git hooks分为客户端hook和服务端hook。客户端hook又分为pre-commit、prepare-commit-msg、commit-msg、post-commit等，主要用于控制客户端git的提交工作流。用户可以在项目根目录的.git目录下面配置使用，也可以配置全局git template用于个人pc上的所有git项目使用。服务端hook又分为pre-receive、post-receive、update，主要在服务端接受提交对象时进行调用。

以上这种采用webhook的形式对git commit进行监控就是一种server端的hook，相当于post-receive。这种方式并不能阻止代码的提交，它只是通过告警的形式来约束用户的行为，但最终不规范的commit message还是被提交到了服务器，不利于后面change log的生成。由于公司代码库权限问题，我们目前只能添加这种post-receive类型的webhook。如大家有更高的代码库权限，可以采用server端pre-receive类型的webhook，直接拒绝不规范的git commit message。只要git commit规范了，我们甚至可以考虑把代码和bug、需求关联等等。

当然这块我们也可以考虑客户端的pre-commit，pre-commit在git add提交之后，然后执行git commit时执行，脚本执行没错就继续提交，反之就会驳回。客户端git hooks位于每个git项目下的隐藏文件.git中的hooks文件夹里。我们可以通过修改这块的配置文件添加我们的规则校验，直接阻止不规范message的提交，也可以通过客户端commit-msg类型的hook进行拦截，把不规范扼杀在萌芽之中。修改每个git项目下面.git目录中的hooks文件大家肯定觉得浪费时间，其实这里可以采用配置全局git template来完成。但是这又会涉及到hooks配置文件同步的问题。hooks配置文件在本地，如何让hooks配置文件的修改能同步到所有使用的项目又成为一个问题。所以使用服务端hook还是客户端hook需要根据具体需求做适当的权衡。

git hook不光可以用来做规范限制，它还可以做更多有意义的事情。一次git commit提交的信息量很大，有作者信息、代码库信息、commit等信息。我们的监控服务就根据作者信息做了git commit的统计，这样不仅可以用来监控commit message的规范性，也可以用来监控大家的工作情况。我们也可以把git commit和相关的bug关联起来，我们查看bug时就可以查看解决这个bug的代码修改，很有利于相关问题的追溯。当然我们用同样的方法也可以把git commit和相关的需求关联起来，比如我们定义一种格式feat *786990（需求的ID），然后在git commit的时候按照这种格式提交，webhook就可以根据这种格式把需求和git commit进行关联，也可以用来追溯某个需求的代码量，当然这个例子不一定合适，但足以证明git hook功能之强大，可以给我们的流程规范带来很大的便利。




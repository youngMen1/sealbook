一个优秀的技术团队一定是对代码质量有着高品质的要求。在代码 Review 方面来看，Review工作是技术成长的最好方式。Keep 做到良好的 Review 首先基于 Commit 的细粒度提交，技术团队内部规定：每次提交不应过大，保持代码功能单一的职责。但整体的提交应该包含一个已完成的完整功能, 未完成的功能不应该 Commit，代码作者在每次提交之前都应该自己过一遍代码，走一遍冒烟测试，确保基本的功能是完整的。

其次 Keep 代码的每个 Commit 都必须经过 Code Review。我们做法是强制在 Phabricator 代码库上限制了不能直接合入，必须经过 Review。

在选取合适 Reviewer 方面，功能改动和扩展时 Reviewer 选择模块原作者或者同一组的同事。复杂功能或牵涉到算法相关的 Review, 通常需要和 Author 一起进行 Review。对于超大型 Diff，本组内同事再一起拉会议室进行 Review。

**iOS 技术**

目前 Keep iOS 团队一共有 9 名成员。从业务上划分为 TC，SU，RT 三大块。TC \(Training Course\) 包含课程列表、参加训练、训练过程和课程表等训练健身的相关功能。SU \(Social & User\) 包含了社区、动态、消息、登录注册、相机、直播等功能。RT \(Running Track\) 目前有户外跑步、跑步机、骑行、跑步路线等。在了解完 iOS 团队情况和业务划分后，本文从 iOS 基本架构模型说起，随后描述技术选型和实现的一些细节，最后从技术展望的角度简单谈谈之后的发展和规划。

**架构模型**

  


Keep iOS 基本架构模型设计如下所示

  


# 参考

keep技术团队:

[https://mp.weixin.qq.com/s/Dtpbwvf4UStzoAURv7PcOQ](https://mp.weixin.qq.com/s/Dtpbwvf4UStzoAURv7PcOQ)


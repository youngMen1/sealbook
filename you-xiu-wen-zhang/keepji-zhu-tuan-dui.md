一个优秀的技术团队一定是对代码质量有着高品质的要求。在代码 Review 方面来看，Review工作是技术成长的最好方式。Keep 做到良好的 Review 首先基于 Commit 的细粒度提交，技术团队内部规定：每次提交不应过大，保持代码功能单一的职责。但整体的提交应该包含一个已完成的完整功能, 未完成的功能不应该 Commit，代码作者在每次提交之前都应该自己过一遍代码，走一遍冒烟测试，确保基本的功能是完整的。

其次 Keep 代码的每个 Commit 都必须经过 Code Review。我们做法是强制在 Phabricator 代码库上限制了不能直接合入，必须经过 Review。

在选取合适 Reviewer 方面，功能改动和扩展时 Reviewer 选择模块原作者或者同一组的同事。复杂功能或牵涉到算法相关的 Review, 通常需要和 Author 一起进行 Review。对于超大型 Diff，本组内同事再一起拉会议室进行 Review。

**iOS 技术**

目前 Keep iOS 团队一共有 9 名成员。从业务上划分为 TC，SU，RT 三大块。TC \(Training Course\) 包含课程列表、参加训练、训练过程和课程表等训练健身的相关功能。SU \(Social & User\) 包含了社区、动态、消息、登录注册、相机、直播等功能。RT \(Running Track\) 目前有户外跑步、跑步机、骑行、跑步路线等。在了解完 iOS 团队情况和业务划分后，本文从 iOS 基本架构模型说起，随后描述技术选型和实现的一些细节，最后从技术展望的角度简单谈谈之后的发展和规划。

**架构模型**

Keep iOS 基本架构模型设计如下所示

![](/static/image/微信图片_20200603141750.jpg)

上图模型从下到上依次分为 Core, Service, Business 三层：

* Core 层主要包含网络请求、基础通用模块、数据存储以及第三方服务模块，这些模块接口直接提供给上层服务。

* Service 层包含了各种包装好的服务给业务层调用，比如本地日志系统、统计埋点以及第三方服务的封装等等。这一层承上启下，连接了 Core 层和 Business 层。

* Business \(SU/TC/RT\) 的解耦以 KEPMedium 模块作为共同依赖组件，其核心实现是采用 Runtime 利用反射 Class 动态调用达到解耦的效果。 需要特别指出的是，由于项目庞大，在 Business 中首先需要将具体业务细化为子业务模块，比如图示的 Timeline 模块和 Live 模块；然后子业务内部再划分 MVC/MVVM ，但业务相关的工具类需要单独抽出；另外， Core 层和 Service 层的部分模块被要求放到私有仓库中，用 Carthage 作为Framework 为日后 Keep 其他 App 提供基础依赖。

**技术实现**

最开始 Keep 项目的 UI 编写主要采用 Storyboard 和 XIB，因为当时正处于苹果积极推崇 StoryBoard 的阶段。直至 2016 年5 月份，由于 Xcode 升级导致了 Storyboard 兼容性上的一些问题，外加项目工程大，导致 Storyboard 打开更耗费系统资源、编译时间更长，因此之后选取了 Masonry 作为 AutoLayout 主要开发库，同时资源图片采用 PDF 矢量图。

**IDE 的选择**

**    
**

Tim Cook 来 Keep 参观时曾询问到 Xcode 能否满足当前的开发需求。作为 Apple 推出的官方 IDE，Xcode 对苹果特有的 Storyboard、XIB、plist 等文件类型均支持地比较全面，但在代码提示、自动补全等功能方面，同专攻 IDE 的 JetBrains 等公司推出的类似工具相比还略显逊色，例如 AppCode。iOS 开发团队也曾尝试过 AppCode，感受是：纯写代码较顺畅，静态检查也超赞，内置的 Git 操作、Refactor 等功能都非常好用；但 AppCode 对 Swift 和 Objective-C 混编、XIB 等支持的并不佳。目前开发团队中编译器的选择主要依据个人喜好，在纯写代码时更加偏向于 AppCode。

**编译的优化**

  


由于需求众多， Keep 项目逐渐庞大，导致编译时间加长，需要对其进行优化，iOS 团队主要从以下几个方面入手：

1、尽可能减少宏的定义，使用静态常量代替。

2、删除无用的头文件引用，采取 @class 或 @import module 方式代替。

3、减少 Storyboard、XIB 的使用，因为 Storyboard 需要编译 Copy 等过程。

4、第三方库尽量使用静态库 Framework。

  


优化过后，整个工程编译时间从 15 分钟缩短至 7 分钟左右，而且随着组件化的细分，相信这个时间还会继续缩短。另外，业务模块相互独立后，Business 中各个业务也将单独拆成独立 Project 进行编译，使业务人员开发时只需编译自己的模块，开发效率更高。

  


**自动化流程**

  


iOS 团队采用 Jenkins + Xcodebuild + Fastlane 的方式，编写了打包上传 App Store、发布内测、灰度的自动化流程工具，并在公司的 Mac Pro 上部署了这一工具。当然除了打包之外，还部署了其他一些自动化服务，比如前台接待系统，办公网络抓包工具等等。

  




「前台接待系统」是 iOS 团队成员在业余时间，用 Swift + Spring Boot + Mysql 开发的一个自动接待工具。快递小哥来了之后只需进行扫码就可用短信通知快递收件人，基本替代了收发快递的工作，前台人员工作量明显减少了。像这种常用工具的搭建在 Keep 技术团队中非常常见，也是探索新技术的一种方式。

# 参考

keep技术团队:

[https://mp.weixin.qq.com/s/Dtpbwvf4UStzoAURv7PcOQ](https://mp.weixin.qq.com/s/Dtpbwvf4UStzoAURv7PcOQ)


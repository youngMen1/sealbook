# 02 | 代码加锁：不要让“锁”事成为烦心事

你好，我是朱晔。

在上一讲中，我与你介绍了使用并发容器等工具解决线程安全的误区。今天，我们来看看解决线程安全问题的另一种重要手段——锁，在使用上比较容易犯哪些错。我先和你分享一个有趣的案例吧。有一天，一位同学在群里说“见鬼了，疑似遇到了一个 JVM 的 Bug”，我们都很好奇是什么 Bug。
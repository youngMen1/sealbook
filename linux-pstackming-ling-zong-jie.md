# 1.linux pstack命令总结

1\). 查看线程数\(比pstree, 包含了详细的堆栈信息\)

2\). 能简单验证是否按照预定的调用顺序/调用栈执行

3\). 采用高频率多次采样使用时, 能发现程序当前的阻塞在哪里, 以及性能消耗点在哪里?

4\). 能反映出疑似的死锁现象\(多个线程同时在wait lock, 具体需要进一步验证\)

## 1.1.pstack的安装

pstack是gdb的一部分，如果系统没有pstack命令，使用yum搜索安装gdb即可

`yum install gdb -y`

## 1.2.pstack 与 gstack 区别

pstack是/usr/bin/gstack的软链接
![](/static/image/20190814071649437.png)

# 2.使用实例

问题：php某进程一直卡着在running，找不到具体原因

![](/static/image/20190814072058602.jpg)

执行 gstack 1696 效果如下：

![](/static/image/2019081407285933.jpg)

可以看到运行堆栈信息已经打印出来，可根据信息排错

## pstack原理

gstack本身是基于gdb封装的shell脚本.

![](/static/image/20190814072922817.png)

让我们简单分析下这个强大的shell脚本:

![](/static/image/20190814072932420.png)


# 3.总结

由于代码太长, 这边选取最核心的片段, backtrace="thread apply all bt"
shell采用了here document的方式, 完成了GDB的交互工作(注意EOF标识, 及范围内的交互命令). 
重要的是输入thread apply all bt这个交互命令. 该命令要求输出所有的线程堆栈信息.
对GDB输出的结果, 通过管道并借助sed命令进行了替换和过滤.

pstack其实是gdb的一个功能封装, 但其实现的功能, 确实非常实用

# 4.参考

[http://lnmp.ailinux.net/pstack](http://lnmp.ailinux.net/pstack)

linux pstack命令总结

[https://www.cnblogs.com/kerrycode/p/5249968.html](https://www.cnblogs.com/kerrycode/p/5249968.html)

[详解命令-pstack](https://www.linuxprobe.com/pstack-command.html)

[https://www.linuxprobe.com/pstack-command.html](https://www.linuxprobe.com/pstack-command.html)


# 1.linux pstack命令总结

1\). 查看线程数\(比pstree, 包含了详细的堆栈信息\)

2\). 能简单验证是否按照预定的调用顺序/调用栈执行

3\). 采用高频率多次采样使用时, 能发现程序当前的阻塞在哪里, 以及性能消耗点在哪里?

4\). 能反映出疑似的死锁现象\(多个线程同时在wait lock, 具体需要进一步验证\)



## 1.1.pstack的安装

pstack是gdb的一部分，如果系统没有pstack命令，使用yum搜索安装gdb即可

`yum install gdb -y`

# 2.总结

# 3.参考

[http://lnmp.ailinux.net/pstack](http://lnmp.ailinux.net/pstack)

linux pstack命令总结

[https://www.cnblogs.com/kerrycode/p/5249968.html](https://www.cnblogs.com/kerrycode/p/5249968.html)

[详解命令-pstack](https://www.linuxprobe.com/pstack-command.html)

[https://www.linuxprobe.com/pstack-command.html](https://www.linuxprobe.com/pstack-command.html)


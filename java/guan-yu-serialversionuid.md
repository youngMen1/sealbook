# 总结

[什么时候需要实现序列化Serializable](https://www.cnblogs.com/aaronRhythm/p/12701021.html)

一般来说如果你的对象需要网络传输或者持久化（对象直接转换为字节的形式传输），只要你的对象需要转换为字节的形式那么你的对象就要实现Serializable接口。比如使用dubbo使用rpc的方式调用接口，那么接口参数就一定要实现Serializable接口。

如果只是转换为字符串的形式与网络打交道，那么就不需要实现Serializable接口。

# 参考

为什么阿里巴巴禁止开发人员修改serialVersionUID 字段的值:

[https://developer.aliyun.com/article/756734?utm\_content=g\_1000126238](https://developer.aliyun.com/article/756734?utm_content=g_1000126238)


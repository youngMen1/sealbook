# 1.MapStruct

首先，我们先说一下MapStruct这类框架适用于什么样的场景，为什么市面上会有这么多的类似的框架。

在软件体系架构设计中，分层式结构是最常见，也是最重要的一种结构。很多人都对三层架构、四层架构等并不陌生。

甚至有人说："计算机科学领域的任何问题都可以通过增加一个间接的中间层来解决，如果不行，那就加两层。"

但是，随着软件架构分层越来越多，那么各个层次之间的数据模型就要面临着相互转换的问题，典型的就是我们可以在代码中见到各种O，如DO、DTO、VO等。

一般情况下，同样一个数据模型，我们在不同的层次要使用不同的数据模型。如在数据存储层，我们使用DO来抽象一个业务实体；在业务逻辑层，我们使用DTO来表示数据传输对象；到了展示层，我们又把对象封装成VO来与前端进行交互。

那么，数据的从前端透传到数据持久化层（从持久层透传到前端），就需要进行对象之间的互相转化，即在不同的对象模型之间进行映射。

## 1.1.MapStruct的使用

## 1.2.MapStruct的性能

# 2.总结

本文介绍了一款Java中的字段映射工具类，MapStruct，他的用法比较简单，并且功能非常完善，可以应付各种情况的字段映射。  
并且因为他是编译期就会生成真正的映射代码，使得运行期的性能得到了大大的提升。

# 3.参考

丢弃掉那些BeanUtils工具类吧，MapStruct真香

[https://mp.weixin.qq.com/s/L\_lMbHuU138NXAV7Sv8moA](https://mp.weixin.qq.com/s/L_lMbHuU138NXAV7Sv8moA)


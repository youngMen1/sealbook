# 1.属性拷贝工具

在日常开发中，我们经常需要给对象进行赋值，通常会调用其set/get方法，有些时候，如果我们要转换的两个对象之间属性大致相同，会考虑使用属性拷贝工具进行。
如我们经常在代码中会对一个数据结构封装成DO、SDO、DTO、VO等，而这些Bean中的大部分属性都是一样的，所以使用属性拷贝类工具可以帮助我们节省大量的set和get操作。
市面上有很多类似的工具类，比较常用的有

1、Spring BeanUtils

2、Cglib BeanCopier

3、Apache BeanUtils

4、Apache PropertyUtils

5、Dozer

## 1.1.性能测试
| 工具类 | 执行1000次耗时 | 执行10000次耗时 | 执行100000次耗时 | 执行1000000次耗时 |
| :--- | :--- | :--- | :--- | :--- |
| Spring BeanUtils | 5ms | 10ms | 45ms | 169ms |
| Cglib BeanCopier | 4ms | 18ms | 45ms | 91ms |
| Apache PropertyUtils | 60ms | 265ms | 1444ms | 11492ms |
| Apache BeanUtils | 138ms | 816ms | 4154ms | 36938ms |
| Dozer | 566ms | 2254ms | 11136ms | 102965ms |

## 1.2.折线图对比
![](/static/image/微信图片_20200810100334.jpg)


# 2.总结
在性能方面，Spring BeanUtils和Cglib BeanCopier表现比较不错，而Apache PropertyUtils、Apache BeanUtils以及Dozer则表现的很不好。
所以，如果考虑性能情况的话，建议大家不要选择Apache PropertyUtils、Apache BeanUtils以及Dozer等工具类。
很多人会不理解，为什么大名鼎鼎的Apache开源出来的的类库性能确不高呢？这不像是Apache的风格呀，这背后导致性能低下的原因又是什么呢？
其实，是因为Apache BeanUtils力求做得完美, 在代码中增加了非常多的校验、兼容、日志打印等代码，过度的包装导致性能下降严重。

本文通过对比几种常见的属性拷贝的类库，分析得出了这些工具类的性能情况，最终也验证了《阿里巴巴Java开发手册》中提到的"Apache BeanUtils 效率低"的事实。
但是本文只是站在性能这一单一角度进行了对比，我们在选择一个工具类的时候还会有其他方面的考虑，比如使用成本、理解难度、兼容性、可扩展性等，对于这种拷贝类工具类，我们还会考虑其功能是否完善等。
就像虽然Dozer性能比较差，但是他可以很好的和Spring结合，可以通过配置文件等进行属性之间的映射等，也受到了很多开发者的喜爱。

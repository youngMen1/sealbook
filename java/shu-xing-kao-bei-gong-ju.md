1、Spring BeanUtils

2、Cglib BeanCopier

3、Apache BeanUtils

4、Apache PropertyUtils

5、Dozer

# 2.性能测试
| 工具类 | 执行1000次耗时 | 执行10000次耗时 | 执行100000次耗时 | 执行1000000次耗时 |
| :--- | :--- | :--- | :--- | :--- |
| Spring BeanUtils | 5ms | 10ms | 45ms | 169ms |
| Cglib BeanCopier | 4ms | 18ms | 45ms | 91ms |
| Apache PropertyUtils | 60ms | 265ms | 1444ms | 11492ms |
| Apache BeanUtils | 138ms | 816ms | 4154ms | 36938ms |
| Dozer | 566ms | 2254ms | 11136ms | 102965ms |

## 折线图对比
![](/static/image/微信图片_20200810100334.jpg)



在性能方面，Spring BeanUtils和Cglib BeanCopier表现比较不错，而Apache PropertyUtils、Apache BeanUtils以及Dozer则表现的很不好。
所以，如果考虑性能情况的话，建议大家不要选择Apache PropertyUtils、Apache BeanUtils以及Dozer等工具类。
很多人会不理解，为什么大名鼎鼎的Apache开源出来的的类库性能确不高呢？这不像是Apache的风格呀，这背后导致性能低下的原因又是什么呢？
其实，是因为Apache BeanUtils力求做得完美, 在代码中增加了非常多的校验、兼容、日志打印等代码，过度的包装导致性能下降严重。

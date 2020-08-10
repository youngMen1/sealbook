1、Spring BeanUtils

2、Cglib BeanCopier

3、Apache BeanUtils

4、Apache PropertyUtils

5、Dozer

|  |
| :--- |


| 1 | 执行1000次耗时 | 执行10000次耗时 | 执行100000次耗时 | 执行1000000次耗时 |
| :--- | :--- | :--- | :--- | :--- |
| Spring BeanUtils | 5ms | 10ms | 45ms | 169ms |
| Cglib BeanCopier | 4ms | 18ms | 45ms | 91ms |
| Apache PropertyUtils | 60ms | 265ms | 1444ms | 11492ms |
| Apache BeanUtils | 138ms | 816ms | 4154ms | 36938ms |
| Dozer | 566ms | 2254ms | 11136ms | 102965ms |




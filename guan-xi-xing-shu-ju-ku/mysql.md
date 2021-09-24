## 1.覆盖索引
假设我们为表 person 的 NAME 和 SCORE 列建了联合索引，
那么下面第二条语句应该可以走索引覆盖，而第一条语句需要回表：

```
explain select * from person where NAME='name1';
explain select NAME,SCORE from person where NAME='name1';
```

## 2.排序无法使用索引的情况
* 1.对于使用联合索引进行排序的场景，多个字段排序ASC和DESC混用；
* 2.a+b作为联合索引，按照a范围查询后按照b排序；
* 3.排序列涉及到的多个字段不属于同一个联合索引；
* 4.排序列使用了表达式；
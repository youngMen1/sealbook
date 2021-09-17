## 覆盖索引
假设我们为表 person 的 NAME 和 SCORE 列建了联合索引，
那么下面第二条语句应该可以走索引覆盖，而第一条语句需要回表：

```
explain select * from person where NAME='name1';
explain select NAME,SCORE from person where NAME='name1';
```


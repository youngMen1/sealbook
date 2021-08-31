# 日常篇

## 1.判断int和Integer类型是否相等时，报空指针异常
需要注意在判断int和Integer是否相等时，会先将Integer拆箱，如果Integer为null，会报空指针异常。

因此需要提前处理好Integer对象为null的情况。

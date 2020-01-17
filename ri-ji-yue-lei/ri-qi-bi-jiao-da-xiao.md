## java中Date日期类型的大小比较
### 方法一:
java.util.Date类实现了Comparable接口，可以直接调用Date的compareTo()方法来比较大小
### 方法二：
通过Date自带的before()或者after()方法比较
### 方法三：
通过调用Date的getTime()方法获取到毫秒数来进行比较
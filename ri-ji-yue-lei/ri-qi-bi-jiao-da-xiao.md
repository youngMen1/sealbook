## java中Date日期类型的大小比较

### 方法一:

java.util.Date类实现了Comparable接口，可以直接调用Date的compareTo\(\)方法来比较大小

```
String beginTime = "2018-07-28 14:42:32";
String endTime = "2018-07-29 12:26:32";

SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

try {
    Date date1 = format.parse(beginTime);
    Date date2 = format.parse(endTime);

   // date1大于date2返回1，date1小于date2返回-1，相等返回0
    int compareTo = date1.compareTo(date2);

    System.out.println(compareTo);

} catch (ParseException e) {
    e.printStackTrace();
}
```

compareTo\(\)方法的返回值，date1小于date2返回-1，date1大于date2返回1，相等返回0

---

### 方法二：

通过Date自带的before\(\)或者after\(\)方法比较

```
String beginTime = "2018-07-28 14:42:32";
String endTime = "2018-07-29 12:26:32";

SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

try {
    Date date1 = format.parse(beginTime);
    Date date2 = format.parse(endTime);

    // date1大于date2返回true，date1小于date2返回false
    boolean before = date1.before(date2);
    boolean before = date1.after(date2);

    System.out.println(before);

} catch (ParseException e) {
    e.printStackTrace();
}
```

before\(\)或者after\(\)方法的返回值为boolean类型

### 方法三：

通过调用Date的getTime\(\)方法获取到毫秒数来进行比较


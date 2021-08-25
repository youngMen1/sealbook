# Java8集合和字符串互转

## 1.使用谷歌的Joiner（代码超级短）

```java
import com.google.common.base.Joiner;
import java.util.ArrayList;
import java.util.List;

public class Convert {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(5);
        list.add(4);
        list.add(1);

        String result = Joiner.on(",").join(list)
        System.out.println(result);

List<Integer> list = Arrays.stream(result.split(",")).map(Integer::parseInt).collect(Collectors.toList())

    }
}
```




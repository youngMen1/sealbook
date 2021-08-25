# Java8集合和字符串互转

## 使用谷歌的Joiner（代码超级短）

```
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
        System.out.println(Joiner.on(",").join(list));
    }
}
```


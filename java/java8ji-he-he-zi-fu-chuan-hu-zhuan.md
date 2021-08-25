# Java8集合和字符串互转

## 1.String 和 List互转

```java
1.使用谷歌的Joiner（代码超级短）
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


2、使用String.join方法（不用需要CharSequence类型的子类才行，并且需要同类型）
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

public class Convert {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(5);
        list.add(4);
        list.add(1);
        System.out.println(String.join(",", list.stream().map(String::valueOf).collect(Collectors.toList())));
    }
}

3、使用collect转换
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

public class Convert {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(5);
        list.add(4);
        list.add(1);
        System.out.println(list.stream().map(String::valueOf).collect(Collectors.joining(",")));
    }
}
```

## 2.String 和 List互转

```java
String auctionBlockIds = "45,46,47";

1.String 转 List<String>

List<String> auctionBlockId= Arrays.asList(auctionBlockIds.split(",")).stream().map(block -> (block.trim())).collect(Collectors.toList());

2.List<String> 转 String 

String auctionBlockIds=String.join(",",auctionBlockId);

3.String 转 List<Integer>

List<Integer> auctionBlockId= Arrays.stream(auctionBlockIds.split(",")).map(s -> Integer.valueOf(s.trim())).collect(Collectors.toList());


4.List<Integer> 转 String 

String requisitionLineIds=list.stream().map(requisitionLine->String.valueOf(requisitionLine.getRequisitionLineId())).collect(Collectors.joining(","));

或者

String result = Joiner.on(",").join(list)
```




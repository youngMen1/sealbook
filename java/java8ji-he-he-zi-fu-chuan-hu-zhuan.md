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

## 2.


```java
String  auctionBlockIds="45,46,47";

1.String 转List<String>

List<String> auctionBlockId= Arrays.asList(auctionBlockIds.split(",")).stream().map(block -> (block.trim())).collect(Collectors.toList());

2.List<String> 转  String 

String auctionBlockIds=String.join(",",auctionBlockId);
3.String 转List<Integer>

List<Integer> auctionBlockId= Arrays.stream(auctionBlockIds.split(",")).map(s -> Integer.valueOf(s.trim())).collect(Collectors.toList());


4.List<Integer> 转 String 
String requisitionLineIds=list.stream().map(requisitionLine->String.valueOf(requisitionLine.getRequisitionLineId())).collect(Collectors.joining(","));

或者
String result = Joiner.on(",").join(list)

```






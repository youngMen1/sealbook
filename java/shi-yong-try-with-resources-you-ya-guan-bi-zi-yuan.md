# 1.使用try-with-resources优雅关闭资源
JDK1.7之后，引入了try-with-resources，使得关闭资源操作无需层层嵌套在finally中，代码简洁不少，本质是一个语法糖，能够使用try-with-resources关闭资源的类，必须实现AutoCloseable接口。

　　1.7版本之前，传统的关闭资源操作如下：


```
public static void main(String[] args){

    FileInputStream fileInputStream = null;
    try {
        fileInputStream = new FileInputStream("file.txt");
        fileInputStream.read();
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        try {
            assert fileInputStream != null;
            fileInputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```



# 4.参考
使用try-with-resources优雅关闭资源：
https://www.cnblogs.com/youtang/p/11441959.html


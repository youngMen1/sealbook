# **1.Debug模式**

| F9 | resume programe 恢复程序 |
| :--- | :--- |
| Alt+F10 | show execution point 显示执行断点 |
| F8 | Step Over 相当于eclipse的f6 跳到下一步 |
| F7 | Step Into 相当于eclipse的f5就是  进入到代码 |
| Alt+shift+F7 | Force Step Into 这个是强制进入代码 |
| Shift+F8 | Step Out  相当于eclipse的f8跳到下一个断点，也相当于eclipse的f7跳出函数 |
| Atl+F9 | Run To Cursor 运行到光标处 |
| ctrl+shift+F9 | debug运行java类 |
| ctrl+shift+F10 | 正常运行java类 |
| alt+F8 | debug时选中查看值 |
| ctrl+N | 快速查找某个类 |

# 2.IDEA常用快捷键

## 2.1.Ctrl

Ctrl + F 在当前文件进行文本查找

Ctrl + R 在当前文件进行文本替换

Ctrl + Z 撤销

Ctrl + Y 删除光标所在行 或 删除选中的行

Ctrl + D 复制光标所在行 或 复制选择内容，并把复制内容插入光标位置下面

Ctrl + W 递进式选择代码块。可选中光标所在的单词或段落，连续按会在原有选中的基础上再扩展选中范围

Ctrl + E 显示最近打开的文件记录列表

Ctrl + Space 基础代码补全，默认在 Windows 系统上被输入法占用

## 2.2.Alt

Alt + F8  在 Debug 的状态下，选中对象，弹出可输入计算表达式调试框，查看该输入内容的调试结果

Alt + Enter  IntelliJ IDEA 根据光标所在问题，提供快速修复选择，光标放在的位置不同提示的结果也不同

Alt + Insert  代码自动生成，如生成对象的 set / get 方法等。建议用lombak插件

## 2.3.Shift

Shift + F3 在查找模式下，查找匹配上一个

Shift + F6 对文件 / 文件夹 重命名

Shift + Tab 取消缩进

Shift + End 选中光标到当前行尾位置

Shift + Home 选中光标到当前行头位置

## 2.4.Ctrl + Alt

Ctrl + Alt + L 格式化代码，可以对当前文件和整个包目录使用

Ctrl + Alt + O 优化导入的类，可以对当前文件和整个包目录使用

Ctrl + Alt + H 调用层次

Ctrl + Alt + S 打开 IntelliJ IDEA 系统设置

Ctrl + Alt + F7 显示使用的地方。寻找被该类或是变量被调用的地方，用弹出框的方式找出来

Ctrl + Alt + Space 类名自动完成

Ctrl + Alt + 左方向键 退回到上一个操作的地方

Ctrl + Alt + 右方向键 前进到上一个操作的地方

## 2.5.Ctrl + Shift

Ctrl + Shift + F 根据输入内容查找整个项目 或 指定目录内文件

Ctrl + Shift + R 根据输入内容替换对应内容，范围为整个项目 或 指定目录内文件

Ctrl + Shift + J 自动将下一行合并到当前行末尾

Ctrl + Shift + Z 取消撤销

Ctrl + Shift + W 递进式取消选择代码块。可选中光标所在的单词或段落，连续按会在原有选中的基础上再扩展取消选中范围

Ctrl + Shift + N 通过文件名定位 / 打开文件 / 目录，打开目录需要在输入的内容后面多加一个正斜杠

Ctrl + Shift + U 对选中的代码进行大 / 小写轮流转换

Ctrl + Shift + / 代码块注释

Ctrl + Shift + Enter 自动结束代码，行末自动添加分号

## 2.6.其他

F2 跳转到下一个高亮错误 或 警告位置

连按两次Shift 弹出 Search Everywhere 弹出层

## 3**.多Application启动查看**

找到.idea文件下的workspace.xml，并找到RunDashboard

```
 <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType" />
      </set>
 </option>
```

## git commit 最左边侧边栏标签转换为模态框
修改教程
打开 Settings -> Version Control -> Commit
![](/static/image/20210601153726915.png)




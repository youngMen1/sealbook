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

## 3**.多Application启动查看**

找到.idea文件下的workspace.xml，并找到RunDashboard

```
 <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType" />
      </set>
    </option>
```




这是IDEA快捷键拆解系列的第十七篇。

本文主要讲解如何利用好IDEA强大的断点调试功能，含快捷键、经验分享等。

### Shortcuts：快捷键

| 快捷键 | 描述 |
| :--- | :--- |
| `Ctrl + F8` | 添加/取消断点，或直接在左侧点击添加 |
| `Ctrl + Shift + F8` | 查看所有断点，为断点添加条件等 |
| `F8` | 执行下一步 |
| `Shift + Alt + F8` | 强制执行下一步 |
| `F9` | 跳到下一个断点，如果没有则直接运行结束 |
| `Alt + F9` | 运行到光标所在处 |
| `Ctrl + Alt + F9` | 强制运行到光标处 |
| `F7` | 进入代码内部 |
| `Shift + F8` | 退出代码内部 |
| `Alt + F10` | 跳转到断点执行处 |
| `Alt + F8` | 表达式求值 |

### Mute Breakpoints：禁用断点

### Condition Breakpoints：条件断点

1. 若光标在断点处，则快捷键为
   `Ctrl + Shift + F8`
2. 若光标不在断点处，可通过查看所有断点来添加条件，快捷键同上
   `Ctrl + Shift + F8`
3. 通过右键点击断点来添加条件

### Evaluate Expression：表达式求值，快捷键`Alt + F8`

### setValue：一般用于动态修改Debug中运行的值

在分析源码的时候，良好的Debug能力可以帮助我们快速的读懂别人的代码。IDEA为开发者们提供了全面的Debug支持，相信熟练掌握后可以大大的提高我们的Debug能力。




这是IDEA快捷键拆解系列的第八篇。

以下是关于Refactor导航项及其每一子项的拆解介绍，其中，加粗部分的选项是博主认为比较重要的。

* Refactor  
  1. **Refactor This （ 重构当前 ）**  
     `Ctrl + Alt + Shift + T`  
  2. **Rename （ 重命名 ）**  
     `Shift + F6`  
  3. Rename File  
  4. Change Signature （ 修改方法、类的签名，含参数、返回值类型等 ）   
     `Ctrl + F6`  
  5. Type Migration （ 类型迁移 ）   
     `Ctrl + Shift + F6`  
  6. Make Static （ 添加Static关键字 ）  
  7. Convert To Instance Method （ 转换为实例方法 ）

  ---

  1. **Move （ 移动文件 ）**
     `F6`
  2. **Copy （ 拷贝文件 ）**
     `F5`
  3. **Safe Detele （ 安全删除，可用在方法上进行快速删除 ）**
     `Alt + Delete`

  ---

  1. Extract（ 提取 ）

     * **Variable （ 变量 ）**
       `Ctrl + Alt + V`
     * **Constant （ 常量 ）**
       `Ctrl + Alt + C`
     * **Filed （ 类字段 ）**
       `Ctrl + Alt + F`
     * **Parameter （ 参数 ）**
       `Ctrl + Alt + p`

     ---

     * Functional Parameter （ 函数式参数 ） 
       `Ctrl + Alt + Shift + P`
     * Parameter Object

     ---

     * **Mehtod （ 方法 ）**
       `Ctrl + Alt + M`
     * Type Parameter
     * Method Object

     ---

     * Delegate
     * Interrface
     * Superclass
     * Subquery ad CTE

  2. Inline （ 转换为内联、方法链形式的调用 ） 
     `Ctrl + Alt + N`
  3. Find and Replace Code Duplicates
  4. Invert Boolean

  ---

  1. Pull Members Up
  2. Push Members Down
  3. Push ITDs In
  4. Use Interface Where Possible
  5. Replace Inheritance with Delegation
  6. Remobe Middleman
  7. Wrap Method Return Value

  ---

  1. Convert Anonymous to Inner
  2. Encapsulate Fields （ 封装字端，用于生成Getter/Setter ）
  3. Replace Temp With Query
  4. Replace Constructor with Factory Method
  5. Replace Constructor with Builder

  ---

  1. Generify
  2. Migrate

  ---

  1. Lombok \( Lombok插件：添加 \)

     * Default @Date

     ---

     * Default @Getter
     * Default @Setter
     * Default @EqualsAndHashcode
     * Default @ToString

     ---

     * @Log \(and friends\)

  2. Delombok \( Lombok插件：删除 \)

     * All lombok annotations

     ---

     * @Data
     * @Value
     * @Wither
     * @Delegate
     * @Builder

     ---

     * @Constructors
     * @Getter
     * @Setter
     * @EqualsAndHashcode
     * @ToString

     ---

     * @Log \(and friends\)

  3. Internationalize（国际化）




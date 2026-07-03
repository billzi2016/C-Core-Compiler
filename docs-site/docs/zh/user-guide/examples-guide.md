# 示例程序导览

## 为什么 `examples/` 不只是“放几个样例”

很多仓库的 `examples/` 目录只是演示用脚本集合，但这个项目里的 `examples/` 更像一组带教学意义的语言样本。

它们至少承担四个角色：

1. 展示当前编译器到底支持哪些语言特性
2. 给 CLI 和端到端流程提供最小可运行输入
3. 帮助读者按难度理解语言能力边界
4. 给编译原理学习提供逐层观察材料

所以看 `examples/` 时，不应该只把它们当成“能跑一下的 demo”，而应该把它们看成：

- 语言子集边界说明
- 编译路径样本集
- 调试与学习路线图

## 为什么同一个主题常有两份示例

这个项目里你会看到很多成对文件：

- `hello.c` 与 `hello_stdout.c`
- `fib.c` 与 `fib_stdout.c`
- `factorial.c` 与 `factorial_stdout.c`

这不是重复劳动，而是有明确目的。

### `return` 型示例

例如：

- `fib.c`
- `factorial.c`

这类程序通常通过 `return` 表达结果。

它们更适合：

- 自动测试
- 退出码验证
- 小而明确的正确性判断

从编译原理角度看，这类示例更偏向“验证语义和执行结果是否正确”，不强调展示效果。

### `stdout` 型示例

例如：

- `fib_stdout.c`
- `fib_sequence_stdout.c`

这类程序通过打印结果来展示行为。

它们更适合：

- 人眼检查
- 教学展示
- 演示程序流程

从学习角度说，这类示例更适合第一次接触项目的人，因为结果更直观。

## 如何按原理理解不同示例

### `hello.c`

这是最小起点。

适合观察：

- 最基础的函数结构
- 最简单的编译链打通
- 最小后端输出长什么样

如果你想第一次看 Token、AST、后端 C，这个示例最合适。

### `factorial.c`

这个示例适合观察：

- 函数调用
- 递归
- 返回路径

如果你要理解函数调用和返回值在 AST、IR、后端中的形态变化，这个例子很有代表性。

### `fib.c`

这个示例适合观察：

- 多次递归调用
- 条件分支
- 表达式组合

它比 `hello.c` 更复杂，但仍然属于基础 `int` 子集范围，是理解 IR 路径的好材料。

### `control_flow_demo.c`

这个示例最适合观察：

- `if / else`
- `while`
- `for`
- 控制流 lowering

如果你想重点看：

- 标签怎么生成
- 跳转怎么形成
- IR 控制流图是什么形状

那么这个例子比看单纯算术更有价值。

### `char_demo.c`

这个示例适合观察：

- `char`
- 字符字面量

它的重要性在于，它通常已经不再只是最基础的 `int` 子集，因此很适合拿来理解“扩展特性为什么会触发另一条后端路径”。

### `string_demo.c`

这个示例适合观察：

- 字符串字面量
- 与高级特性相关的后端输出

如果你对“为什么当前项目不是所有特性都走 IR”还不够敏感，这个示例会很有帮助。

### `array_demo.c`

这个示例适合观察：

- 定长数组声明
- 下标访问

从原理上讲，它特别适合帮助你理解：

- 数组这种结构为什么让简单 IR 统一化变得更复杂
- 为什么当前项目会选择先走 AST Backend

### `pointer_demo.c`

这个示例适合观察：

- 指针声明
- `&` 取地址
- `*` 解引用

它是当前项目里非常关键的一类示例，因为它直接碰到了更真实的内存模型问题。

如果你想理解“为什么第一代没有急着把所有复杂特性压进 IR”，这个示例几乎是最佳材料。

## 哪些示例适合看 IR，哪些更适合看 AST Backend

### 更适合看 IR 的示例

优先看这些：

- `hello.c`
- `factorial.c`
- `fib.c`
- `control_flow_demo.c`

因为它们更贴近基础 `int` 子集主路径：

```text
AST -> IR -> 优化 -> Backend C
```

### 更适合看 AST Backend 的示例

优先看这些：

- `char_demo.c`
- `string_demo.c`
- `array_demo.c`
- `pointer_demo.c`

因为它们更容易触发当前项目的扩展特性路径：

```text
AST -> AST Backend -> Backend C
```

这组示例非常适合用来理解当前架构为什么是“两条后端路径”。

## 如果你的目标是学习编译原理，应该怎么读这些示例

这里给一个很实用的顺序。

### 第一步：先看最小例子

先看：

- `hello.c`

然后依次观察：

- Token
- AST
- 后端 C
- 最终可执行文件

先建立“最小链路”的感觉。

### 第二步：再看控制流

继续看：

- `control_flow_demo.c`

重点观察：

- AST 中控制流节点如何组织
- IR 中标签和跳转如何出现
- DOT 图里的边怎么形成

### 第三步：再看函数和递归

继续看：

- `factorial.c`
- `fib.c`

重点观察：

- 调用结构
- 返回路径
- 条件与递归组合

### 第四步：最后看扩展特性

再看：

- `char_demo.c`
- `string_demo.c`
- `array_demo.c`
- `pointer_demo.c`

重点理解：

- 它们为什么比基础 `int` 子集更难统一进 IR
- 它们为什么更适合先走 AST Backend

## 结合 CLI 看示例时，最推荐的方式

不要只运行最终程序。

最推荐的是针对同一个示例按顺序跑：

```bash
python3 -m c_core_compiler examples/hello.c --emit-tokens
python3 -m c_core_compiler examples/hello.c --emit-ast
python3 -m c_core_compiler examples/hello.c --emit-ir
python3 -m c_core_compiler examples/hello.c --emit-c
```

然后换成 `control_flow_demo.c`、`pointer_demo.c` 再重复一次。

这样你不是在“看很多示例”，而是在“看同一套编译器如何处理不同语言结构”。

## 一句话总结

这个项目里的 `examples/` 不是零散示例集合，而是一套帮助你理解语言子集边界、编译路径差异和各阶段输出形态的学习样本库。

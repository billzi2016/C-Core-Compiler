# 快速开始

## 这篇文档适合谁

如果你第一次接触这个项目，最容易卡住的问题通常不是“编译原理”，而是：

- 先跑哪个命令
- 怎么确认项目真的工作了
- 哪些示例最适合入门
- 应该先看源码、看文档，还是看中间输出

这篇文档就是先把这些最容易卡人的地方摊开讲，不跟你兜圈子。

## 先做什么

建议第一步先跑测试，确认当前环境能正常使用这个项目：

```bash
python3 -m unittest discover -s tests -v
```

先跑测试，不是因为“规范流程应该这样”，而是因为这一步最省时间。它先帮你确认：

- Python 环境本身没问题
- 当前代码状态至少还在测试能兜住的范围里
- 如果后面跑示例翻车，锅更可能在工具链或本地环境，而不是项目已经先坏了

## 第一个建议运行的示例

最小入门建议从 `examples/hello.c` 开始：

```bash
python3 -m c_core_compiler examples/hello.c -o build/hello
./build/hello
```

这个例子适合拿来热身，因为：

- 语法简单
- 编译链最短
- 出错时定位范围小

如果这个例子都跑不通，就别急着往数组、指针、字符串那边冲，不然你只会多出一堆噪音，问题还更难定位。

## 如果你想直接看到输出

很多基础示例是通过进程退出码表达结果的，不是直接打印到终端。

如果你更想“一眼看到结果”，建议先跑 `_stdout` 系列：

```bash
python3 -m c_core_compiler examples/fib_stdout.c -o build/fib_stdout
./build/fib_stdout
```

或者直接看更明显的序列输出：

```bash
python3 -m c_core_compiler examples/fib_sequence_stdout.c -o build/fib_sequence_stdout
./build/fib_sequence_stdout
```

## 建议怎样理解输出结果

这个项目里结果大致有两种表达方式，读的时候别混了。

### 1. `return` 型

程序通过退出码表达结果。

这种方式更适合拿来：

- 自动化测试
- 脚本验证
- 小型数值正确性检查

### 2. `stdout` 型

程序把结果直接打印出来。

这种方式更适合拿来：

- 人眼检查
- 展示效果
- 教学演示

## 初学者最推荐的阅读顺序

如果你想边跑边理解项目，建议按这个顺序：

1. 先看 [文档首页](index.md)
2. 再看 [这个编译器是怎么做的](project/how-this-compiler-works.md)
3. 然后看 [整体架构与两条编译路径](project/compiler-overview.md)
4. 跑 `--emit-tokens`、`--emit-ast`、`--emit-c` 看中间结果
5. 最后再深入到各阶段实现文档

## 最有用的几个调试命令

输出 Token：

```bash
python3 -m c_core_compiler examples/hello.c --emit-tokens
```

输出 AST：

```bash
python3 -m c_core_compiler examples/hello.c --emit-ast
```

输出 IR：

```bash
python3 -m c_core_compiler examples/hello.c --emit-ir
```

输出后端 C：

```bash
python3 -m c_core_compiler examples/hello.c --emit-c
```

这些命令真正值钱的地方在于：你不用每次都傻等最终程序跑出来才知道对不对，而是能一路看到编译器中途到底做了什么。

## 如果你下一步要做什么

根据你的目标，建议继续看：

- 想理解项目原理：看 [项目文档](project/how-this-compiler-works.md)
- 想理解仓库怎么组织：看 [仓库结构导览](project/repository-map.md)
- 想系统理解命令行调试入口：看 [CLI 使用说明](user-guide/cli-guide.md)
- 想理解各个示例分别在验证什么：看 [示例程序导览](user-guide/examples-guide.md)
- 想理解怎么验证行为：看 [测试与验证](user-guide/testing-and-validation.md)

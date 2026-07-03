# 仓库结构导览

## 为什么要单独讲目录结构

很多人第一次进一个编译器仓库，会立刻遇到两个问题：

- 我应该先看哪个文件
- 哪些目录是核心实现，哪些只是配套材料

这篇文档就是为了解决这个问题。

## 顶层目录怎么看

当前仓库最重要的几部分是：

- `src/c_core_compiler/`
- `tests/`
- `examples/`
- `docs-site/`
- 若干顶层说明文档

你可以把它们理解成四类内容：

- 编译器实现
- 测试验证
- 示例程序
- 项目文档

## `src/c_core_compiler/` 是真正的实现核心

这里放的是编译器主代码。

如果按编译阶段来看，重点文件包括：

- `tokens.py`：Token 定义
- `lexer.py`：词法分析
- `ast_nodes.py`：AST 节点定义
- `parser.py`：语法分析
- `symbol_table.py`：符号表
- `semantic.py`：语义分析
- `ir.py`：IR 结构
- `ir_builder.py`：AST 到 IR 的 lowering
- `optimizer.py`：保守优化
- `codegen/`：后端代码生成
- `toolchain.py`：系统工具链调用
- `pipeline.py`：统一编译流程封装
- `cli.py`：命令行入口

如果你只想理解“主链路”，优先看这几个：

- `pipeline.py`
- `cli.py`
- `lexer.py`
- `parser.py`
- `semantic.py`
- `ir_builder.py`
- `codegen/_portable_c.py`

## `tests/` 告诉你项目把哪些行为看作稳定边界

如果你想知道“哪些行为是作者明确想固定住的”，测试目录比 README 更直接。

这里大致对应：

- 词法测试
- 语法与 AST 测试
- 语义测试
- IR 测试
- 后端测试
- CLI 测试
- 端到端测试

读测试时，不只是看“有没有通过”，更要看“项目认为什么行为必须稳定”。

## `examples/` 是最直观的输入样本

这个目录里放的是示例 C 程序。

它们的用途包括：

- 给 CLI 提供最小可运行输入
- 给端到端测试和人工演示提供样本
- 帮助理解当前语言子集到底支持到了哪里

如果你对某个能力是否支持没有把握，先看这里通常比先猜代码更快。

## `docs-site/` 是文档工程目录

这里放的是 MkDocs 文档站点相关内容，包括：

- 文档页面
- 站点配置
- 依赖定义
- 文档建设 PRD

这部分和编译器核心实现分离，是为了让文档系统独立维护。

## 顶层文档文件分别干什么

仓库根目录下还有一些说明文件，例如：

- `README.md` / `README_CN.md`
- `ARCHITECTURE.md`
- `TESTING.md`
- `DEVELOPMENT.md`
- `RELEASE.md`
- `PRD.md`

可以这样理解：

- `README*`：对外总入口
- `ARCHITECTURE.md`：较短的架构摘要
- `TESTING.md`：测试策略摘要
- `DEVELOPMENT.md`：开发相关说明
- `RELEASE.md`：版本说明
- `PRD.md`：项目目标与需求背景

## 如果你第一次读源码，推荐顺序

建议不要从随机文件开始翻。

更好的顺序通常是：

1. 先看 `README.md` 或 `README_CN.md`
2. 看 `docs-site/` 里的项目总览文档
3. 看 `pipeline.py`
4. 再按 `lexer -> parser -> semantic -> ir_builder -> backend` 往下读
5. 最后回头看测试

这样最不容易迷路。

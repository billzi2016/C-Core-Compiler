# 这个编译器是怎么做的

这篇文档现在作为项目文档入口页，不再承担所有细节说明。

如果你想快速理解这个编译器，建议按下面顺序阅读：

1. [整体架构与两条编译路径](./compiler-overview.md)
2. [词法分析与 Token](./lexer-and-tokens.md)
3. [语法分析与 AST](./parser-and-ast.md)
4. [语义分析与符号表](./semantic-analysis.md)
5. [IR 与 Lowering](./ir-and-lowering.md)
6. [优化器做了什么](./optimizer.md)
7. [后端与系统工具链](./backend-and-toolchain.md)
8. [如何调试整条编译流水线](./debugging-the-pipeline.md)

## 先抓住三个核心判断

### 1. 这不是“直接输出机器码”的编译器

这个项目第一代先输出规范化 C，再调用系统里的 `clang` 或 `cc` 生成可执行文件。

这样做的目标不是偷懒，而是先把完整、可验证、可维护的编译链路打通。

### 2. 它不是一条完全统一的单路径编译器

当前实现有两条后端路径：

- 基础 `int` 子集：`AST -> IR -> 优化 -> 规范化 C`
- 扩展能力如 `char`、字符串、数组、指针：`AST -> AST Backend -> 规范化 C`

也就是说，扩展特性目前还没有全部进入统一 IR。

### 3. 这个项目更像工程化迭代，而不是一次性交作业

它的优先级很明确：

- 先把链路做完整
- 再把结构做清楚
- 再保证每一层都可调试、可测试
- 最后再继续向统一 IR、更强优化、原生后端演进

## 一眼看懂总流程

```text
源代码
  -> Lexer
  -> Tokens
  -> Parser
  -> AST
  -> Semantic Analyzer
  -> IR Builder（仅基础 int 子集）
  -> Optimizer
  -> Backend C Generator
  -> clang / cc
  -> 可执行文件
```

如果程序使用了更高级的特性，例如 `char`、数组或指针，那么会跳过现有 IR 路径，改走 AST 直出后端。

## 不要把这条链当成“名词列表”

上面这一串如果只是背名字，其实没有意义。真正要理解的是：每一层都在把“上一层不方便处理的东西”，改造成“下一层更方便处理的东西”。

下面用一段极小的程序来串起来看：

```c
int add(int a, int b) {
    return a + b;
}
```

### 1. 源代码

这一层只是普通文本，也就是字符序列。

编译器这时看到的本质上还是：

```text
i n t 空格 a d d ( i n t 空格 a , ...
```

问题在于，后面的阶段不适合一直在原始字符上做判断。否则每一步都要重复问：

- 这里是关键字还是变量名？
- 这个 `+` 是运算符还是别的字符？
- 这个 `)` 到底属于哪一层结构？

所以第一步必须先切词。

### 2. Lexer

这一层对应源码文件：

- `src/c_core_compiler/lexer.py`

它做的事情不是“理解程序”，而是沿着字符流往前扫，把能识别的片段切出来。

你可以把它想成：

- 看见字母，就尝试扫一个标识符或关键字
- 看见数字，就扫一个整数
- 看见 `'` 或 `"`，就扫字面量
- 看见 `==`、`!=` 这种双字符运算符，优先识别成一个整体
- 看见空白和注释，就跳过

所以 Lexer 的工作方式更像“扫描器”，不是“解释器”。

### 3. Tokens

这是 Lexer 的输出。

上面那段 `add` 代码在这个阶段，大致会变成：

```text
KW_INT("int")
IDENTIFIER("add")
LPAREN("(")
KW_INT("int")
IDENTIFIER("a")
COMMA(",")
KW_INT("int")
IDENTIFIER("b")
RPAREN(")")
LBRACE("{")
KW_RETURN("return")
IDENTIFIER("a")
PLUS("+")
IDENTIFIER("b")
SEMICOLON(";")
RBRACE("}")
EOF
```

这一步的关键价值是：后面不再面对杂乱字符，而是面对“已经分类好的符号序列”。

### 4. Parser

这一层对应源码文件：

- `src/c_core_compiler/parser.py`

它做的事是：把一串 Token 按语法规则拼成结构。

例如它要识别：

- 顶层这里是不是函数定义
- 参数列表从哪里开始，到哪里结束
- `return a + b;` 这里是不是一条返回语句
- `a + b` 这里是不是一个二元表达式

所以 Parser 的核心不是“继续切词”，而是“根据语法规则组装结构”。

### 5. AST

这是 Parser 的输出。

到了这一步，程序不再只是线性序列，而是一棵树。上面那段代码在概念上会变成：

```text
Program
  FunctionDecl add
    params: a, b
    body:
      ReturnStmt
        BinaryExpr(+)
          Name(a)
          Name(b)
```

这就是 AST 的意义：它把“文本顺序”变成“结构关系”。

如果没有 AST，后面的语义分析和 IRBuilder 就要自己重新猜：

- 哪些 Token 构成一个函数
- 哪些 Token 构成一个表达式
- 哪个 `+` 的左右操作数分别是谁

那会非常乱。

### 6. Semantic Analyzer

这一层对应源码文件：

- `src/c_core_compiler/semantic.py`
- `src/c_core_compiler/symbol_table.py`

这一层不再问“语法像不像”，而开始问“语义是否合法”。

比如它会检查：

- `a` 和 `b` 是不是已定义参数
- 有没有重名参数
- `return` 有没有返回值
- 函数调用时参数个数对不对

也就是说，语法分析只保证“长得像程序”，语义分析才保证“这个程序在当前规则下说得通”。

### 7. IR Builder（仅基础 int 子集）

这一层对应源码文件：

- `src/c_core_compiler/ir_builder.py`

它会把 AST 继续往低层展开，变成更偏动作的表示。

对 `return a + b;` 这种很小的语句，概念上会更接近：

```text
t0 = load a
t1 = load b
t2 = binary + t0 t1
return t2
```

注意这里已经不再强调“这是一棵语法树”，而是在强调：

- 先取什么值
- 再做什么运算
- 最后把什么返回

这就是 lowering 的本质。

### 8. Optimizer

这一层对应源码文件：

- `src/c_core_compiler/optimizer.py`

它不会重新发明程序逻辑，而是在已有 IR 上做保守整理。

例如：

- 某个临时变量如果确定是常量，就直接传播
- 某个表达式如果能提前算出来，就折叠
- 某个分支如果根本走不到，就删掉

所以优化器做的不是“重新编译一遍”，而是“在不改变语义的前提下把 IR 整理得更紧凑”。

### 9. Backend C Generator

这一层对应源码文件：

- `src/c_core_compiler/codegen/_portable_c.py`

它的任务是把 IR 或 AST Backend 的结果，生成为规范化 C。

还是以 `return a + b;` 为例，最终后端输出不会再是 AST 或 IR，而会接近：

```c
int add(int a, int b) {
    int r0, r1, r2;
    r0 = a;
    r1 = b;
    r2 = (r0 + r1);
    return r2;
}
```

这不是人手写最自然的 C，但它非常适合：

- 调试
- 对照 IR
- 交给系统编译器继续处理

### 10. `clang / cc`

这一层对应源码文件：

- `src/c_core_compiler/toolchain.py`

这时已经不是项目自己的编译逻辑了，而是把生成的 C 交给系统编译器。

系统工具链负责：

- 真正的 C 编译
- 目标文件生成
- 链接
- 平台相关细节

所以项目本身并没有直接处理机器码和链接细节，而是借用了系统已有能力。

### 11. 可执行文件

这是整条链路的最终产物。

这一步不是“文档里的抽象结果”，而是真正可以运行的程序。例如：

```bash
python3 -m c_core_compiler examples/hello.c -o build/hello
./build/hello
```

这里的 `build/hello` 就是整条链路的最终落点。

## 最关键的一句话

这条编译链不是“把代码重复表示很多遍”，而是每一层都在做一件很具体的事：

- 字符太乱，就先切成 Token
- Token 还是线性的，就组装成 AST
- AST 还太高层，就降成 IR
- IR 太粗糙，就做保守优化
- 优化后的结果还不能直接运行，就生成规范化 C
- 最后交给系统工具链变成真正二进制

理解了这一点，前面那串名词才不再只是名词。

## 这组文档的阅读方式

这套文档不是只讲“概念”，也不是只讲“代码文件名”，而是把两件事结合起来：

- 编译原理里每一层在解决什么问题
- 这个项目的代码到底是怎样实现这些层的

如果你是第一次看编译器，建议按顺序读。

如果你已经懂基础概念，只想快速对照代码结构，可以先看：

- [整体架构与两条编译路径](./compiler-overview.md)
- [IR 与 Lowering](./ir-and-lowering.md)
- [后端与系统工具链](./backend-and-toolchain.md)

## 当前项目最值得注意的工程特点

- 词法分析器和语法分析器都是手写的，控制流很直白
- 语义分析明确依赖符号表做名字和作用域管理
- `int` 子集有一条比较完整的 AST -> IR -> 优化 -> 后端链路
- 扩展特性已经可用，但暂时走 AST 直出后端
- CLI 支持 `--emit-*` 调试模式，适合逐层观察结果

## 一句话总结

这个项目最核心的价值，不是“它已经支持多少 C 特性”，而是“它已经把一个可解释、可验证、可演进的编译器骨架搭出来了”。

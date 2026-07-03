# C-Core-Compiler Docs

> A documentation hub for a staged, explainable C subset compiler built in Python.

This site is designed for two kinds of readers:

- people who want to **use** the compiler and run examples quickly
- people who want to **study** how the compiler is structured internally

<div class="doc-hero-grid" markdown>

<div class="doc-hero-panel" markdown>

### Start In English

Follow the English reading path if you want architecture, implementation, CLI, and examples explained stage by stage.

[Open English docs](en/index.md){ .md-button .md-button--primary }

</div>

<div class="doc-hero-panel" markdown>

### 从中文开始

如果你想用中文阅读项目原理、CLI、示例和仓库结构，可以从这里进入。

[打开中文文档](zh/index.md){ .md-button .md-button--primary }

</div>

</div>

## What You Can Read Here

<div class="doc-card-grid" markdown>

<div class="doc-card" markdown>

### Compiler Architecture

Understand the full pipeline from source code to executable, including why this project currently uses two backend paths.

- [How This Compiler Works](en/project/how-this-compiler-works.md)
- [整体架构与两条编译路径](zh/project/compiler-overview.md)

</div>

<div class="doc-card" markdown>

### Stage-by-Stage Internals

Follow the compiler as it moves through lexing, parsing, semantic analysis, IR lowering, optimization, and backend generation.

- [Lexer and Tokens](en/project/lexer-and-tokens.md)
- [IR 与 Lowering](zh/project/ir-and-lowering.md)

</div>

<div class="doc-card" markdown>

### Practical Debugging

Use the CLI as a structured inspection tool, not just as a launcher. Learn which flag maps to which compiler layer.

- [CLI Guide](en/user-guide/cli-guide.md)
- [CLI 使用说明](zh/user-guide/cli-guide.md)

</div>

<div class="doc-card" markdown>

### Example Programs

Read the sample programs as a learning set for language boundaries, control flow, recursion, arrays, strings, and pointers.

- [Examples Guide](en/user-guide/examples-guide.md)
- [示例程序导览](zh/user-guide/examples-guide.md)

</div>

</div>

## Recommended Reading Paths

=== "For Readers Who Want to Learn the Compiler"

    1. Open the English or Chinese home page
    2. Read the compiler overview
    3. Follow lexing, parsing, semantics, and IR in order
    4. Use the CLI guides to inspect intermediate outputs
    5. Use the examples guide to compare the `int` subset path and the AST backend path

=== "For Readers Who Want to Run It Quickly"

    1. Start with [Getting Started](en/getting-started.md) or [快速开始](zh/getting-started.md)
    2. Run `hello.c`
    3. Run a `_stdout` example
    4. Inspect `--emit-c` to see the generated backend code
    5. Return to the architecture pages when you want to understand why it works

## Project Links

- [GitHub repository](https://github.com/billzi2016/C-Core-Compiler)
- [Docs workflow on GitHub](https://github.com/billzi2016/C-Core-Compiler/actions/workflows/docs.yml)
- [Workflow source](https://github.com/billzi2016/C-Core-Compiler/blob/main/.github/workflows/docs.yml)
- [Documentation PRD for MkDocs](https://github.com/billzi2016/C-Core-Compiler/blob/main/docs-site/mkdocs_prd.md)
- [Documentation PRD for GitHub Actions](https://github.com/billzi2016/C-Core-Compiler/blob/main/docs-site/github_action_prd.md)

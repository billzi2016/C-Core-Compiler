# How This Compiler Works

This page is now the entry point to the project documentation rather than the place that carries every detail by itself.

If you want to understand the compiler step by step, read in this order:

1. [Architecture Overview and the Two Compilation Paths](./compiler-overview.md)
2. [Lexing and Tokens](./lexer-and-tokens.md)
3. [Parsing and the AST](./parser-and-ast.md)
4. [Semantic Analysis and the Symbol Table](./semantic-analysis.md)
5. [IR and Lowering](./ir-and-lowering.md)
6. [What the Optimizer Actually Does](./optimizer.md)
7. [Backend and System Toolchain](./backend-and-toolchain.md)
8. [How to Debug the Whole Pipeline](./debugging-the-pipeline.md)

## Three points to understand first

### 1. This is not a compiler that goes straight to machine code

The first generation emits normalized C first, then invokes the system `clang` or `cc`.

That is not a shortcut for its own sake. It is a way to establish a complete, testable, and maintainable pipeline first.

### 2. It does not have one fully unified internal path yet

The current implementation has two backend paths:

- Basic `int` subset: `AST -> IR -> optimization -> normalized C`
- Extended features such as `char`, strings, arrays, and pointers: `AST -> AST Backend -> normalized C`

So the extended features are usable, but they are not all lowered into one shared IR yet.

### 3. This project behaves like an engineering iteration

Its priorities are clear:

- complete the pipeline first
- keep the structure understandable
- make each layer observable and testable
- then continue toward a more unified IR, stronger optimization, and a lower-level backend

## The pipeline at a glance

```text
source code
  -> Lexer
  -> Tokens
  -> Parser
  -> AST
  -> Semantic Analyzer
  -> IR Builder (basic int subset only)
  -> Optimizer
  -> Backend C Generator
  -> clang / cc
  -> executable
```

If the program uses more advanced features such as `char`, arrays, or pointers, it skips the current IR path and goes through the AST-oriented backend instead.

## How to use this document set

These docs are meant to combine two things:

- what each compiler stage is supposed to do in compiler theory
- how this repository actually implements that stage

If you are new to compilers, read them in order.

If you already know the basics and only want a quick mapping from concepts to code, start with:

- [Architecture Overview and the Two Compilation Paths](./compiler-overview.md)
- [IR and Lowering](./ir-and-lowering.md)
- [Backend and System Toolchain](./backend-and-toolchain.md)

## The most important engineering traits of the current project

- The lexer and parser are both handwritten and intentionally direct
- Semantic analysis explicitly relies on a symbol table for names and scopes
- The `int` subset has a relatively complete AST -> IR -> optimization -> backend pipeline
- Extended features already work, but still go through an AST backend path
- The CLI exposes `--emit-*` inspection modes, which makes the pipeline easy to observe

## One-sentence summary

The main value of this project is not just how many C features it supports. It is that it already has a compiler skeleton that is explainable, testable, and ready to evolve.

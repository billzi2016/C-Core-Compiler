# Architecture Overview and the Two Compilation Paths

## What this page covers

This page stays at the architectural level. It answers three questions:

- How many major layers the compiler has
- How data flows from source code to an executable
- Why the current implementation has two backend paths

## A compiler is not just "code translation"

People often summarize a compiler as "something that translates a high-level language into a low-level one."

That is true, but too coarse to be useful for implementation. Real compilers almost never translate everything in one jump. They move through several representations because each layer solves a different problem:

- the lexical layer decides how characters become tokens
- the syntactic layer decides how tokens become program structure
- the semantic layer decides whether that structure is valid
- the middle layer rewrites high-level structure into forms the backend can handle more directly
- the backend emits code that can actually be built

This project is organized around exactly that idea.

## The main data flow in this project

For the basic `int` subset, the main path is:

```text
source code
  -> Lexer
  -> Tokens
  -> Parser
  -> AST
  -> Semantic Analyzer
  -> IR Builder
  -> Optimizer
  -> Portable C Backend
  -> clang / cc
  -> executable
```

If the program uses more advanced features such as:

- `char`
- strings
- arrays
- pointers

then the path changes to:

```text
source code
  -> Lexer
  -> Tokens
  -> Parser
  -> AST
  -> Semantic Analyzer
  -> AST Backend
  -> clang / cc
  -> executable
```

## Why there are two paths

The reason is straightforward: the current IR is mainly designed for the first-generation basic `int` subset.

As soon as the program uses things like:

- non-`int` base types
- pointer levels
- arrays and indexing
- string literals
- address-of and dereference

the compiler switches to the AST-oriented backend path instead of lowering everything into the current IR.

That is not a design mistake. It is a staged engineering decision:

- finish the main path first
- make the more complex features usable
- only then decide when to unify them under IR

## Why the first generation emits normalized C

The current backend does not generate native assembly. It generates normalized C.

That has several immediate benefits:

- it is easier to make work across platforms
- it can reuse an existing system compiler
- the intermediate result is easier to read
- it works well for testing and teaching

"Normalized C" here does not mainly mean pretty code. It means the generated form is more regular and explicit:

- control flow is made more explicit
- temporary values are named directly
- the backend relies less on high-level syntactic sugar

## Rough module layout

At the code level, the project is roughly divided like this:

- `lexer.py`: lexical analysis
- `parser.py`: recursive-descent parsing
- `ast_nodes.py`: AST data structures
- `semantic.py`: semantic checks
- `symbol_table.py`: scope and symbol management
- `ir.py`: IR data structures
- `ir_builder.py`: AST-to-IR lowering
- `optimizer.py`: conservative optimization
- `codegen/`: backend code generation
- `toolchain.py`: system toolchain invocation
- `pipeline.py`: one shared compilation pipeline
- `cli.py`: command-line entry point

## Why `pipeline.py` matters

Many projects let the CLI, tests, and scripts all rebuild the compilation flow separately.

This project explicitly centralizes the flow in `pipeline.py`. That helps because:

- the CLI does not have to duplicate the phase wiring
- tests do not have to reassemble the pipeline themselves
- the compilation flow is easier to maintain consistently

That is a practical engineering decision, not just a code-style preference.

## The main architectural strengths right now

- each stage has a relatively clear responsibility
- the main path can be inspected stage by stage
- the frontend/backend boundary is explicit
- the backend can evolve later without rewriting the frontend

That is why the project already looks like something that can keep evolving, even though it is not yet a fully unified compiler for every supported feature.

## What to read next

If you want to keep going, the recommended order is:

1. [Lexing and Tokens](./lexer-and-tokens.md)
2. [Parsing and the AST](./parser-and-ast.md)
3. [Semantic Analysis and the Symbol Table](./semantic-analysis.md)
4. [IR and Lowering](./ir-and-lowering.md)

Understanding the frontend and middle layer first makes the later optimizer and backend pages much easier to follow.

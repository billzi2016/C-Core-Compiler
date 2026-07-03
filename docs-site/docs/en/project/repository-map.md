# Repository Map

## Why the repository structure deserves its own page

When people first enter a compiler repository, they usually hit two questions immediately:

- which file should I read first?
- which directories are core implementation and which are support material?

This page is meant to answer that.

## How to read the top-level structure

The most important parts of the repository are:

- `src/c_core_compiler/`
- `tests/`
- `examples/`
- `docs-site/`
- several top-level documentation files

You can think of them as four categories:

- compiler implementation
- validation and testing
- example programs
- project documentation

## `src/c_core_compiler/` is the real implementation core

This is where the compiler itself lives.

If you map it by compilation stage, the key files include:

- `tokens.py`: token definitions
- `lexer.py`: lexical analysis
- `ast_nodes.py`: AST node definitions
- `parser.py`: parsing
- `symbol_table.py`: symbol table
- `semantic.py`: semantic analysis
- `ir.py`: IR structures
- `ir_builder.py`: AST-to-IR lowering
- `optimizer.py`: conservative optimization
- `codegen/`: backend code generation
- `toolchain.py`: system toolchain invocation
- `pipeline.py`: shared compilation pipeline
- `cli.py`: command-line entry point

If you only want the main path first, prioritize:

- `pipeline.py`
- `cli.py`
- `lexer.py`
- `parser.py`
- `semantic.py`
- `ir_builder.py`
- `codegen/_portable_c.py`

## `tests/` tells you what the project considers stable behavior

If you want to know which behaviors the project explicitly tries to preserve, the test directory is often more direct than the README.

It roughly covers:

- lexing tests
- parsing and AST tests
- semantic tests
- IR tests
- backend tests
- CLI tests
- end-to-end tests

When reading tests, look not only at whether they pass, but at what they imply the project considers part of the stable contract.

## `examples/` are the most direct input samples

This directory contains example C programs.

They are useful for:

- minimal CLI inputs
- end-to-end demonstrations
- understanding the actual supported language subset

If you are unsure whether a feature is supported, looking here is usually faster than guessing from memory.

## `docs-site/` is the documentation project

This is the MkDocs documentation site directory. It contains:

- documentation pages
- site configuration
- dependency definitions
- documentation PRDs

It is intentionally separated from the compiler implementation so the docs system can be maintained independently.

## What the top-level documentation files do

At the repository root, there are several explanation files, such as:

- `README.md` / `README_CN.md`
- `ARCHITECTURE.md`
- `TESTING.md`
- `DEVELOPMENT.md`
- `RELEASE.md`
- `PRD.md`

One useful way to think about them is:

- `README*`: public entry point
- `ARCHITECTURE.md`: short architecture summary
- `TESTING.md`: short testing strategy summary
- `DEVELOPMENT.md`: development notes
- `RELEASE.md`: release notes
- `PRD.md`: project goals and requirements context

## A good first-pass reading order

It is usually better not to start by randomly opening files.

A more reliable order is:

1. start with `README.md` or `README_CN.md`
2. read the high-level docs under `docs-site/`
3. inspect `pipeline.py`
4. then read `lexer -> parser -> semantic -> ir_builder -> backend`
5. finally circle back to the tests

That order makes it much easier not to get lost.
